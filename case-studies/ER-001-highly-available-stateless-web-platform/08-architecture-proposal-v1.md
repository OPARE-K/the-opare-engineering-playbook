# ER-001: Architecture Proposal v1

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 08 — Architecture Proposal v1  
**Status:** Proposed — subject to architecture review  
**Date:** 2026-06-28  
**Supersedes:** None (first proposal)

---

## Proposal Disclaimer

This is **architecture proposal v1**, not final design. It is based on [discovery documents 01–07](07-architecture-discovery-summary.md) with **critical gaps still unresolved** (peak load, budget envelope, integration scope, payment boundary). Several choices are **reasonable defaults** pending validation — not proven optima.

The proposal intentionally favours **simplicity and operability** for a small team over maximum resilience. It may not satisfy stricter interpretations of "high availability" until stakeholders confirm targets and budget.

---

## 1. Architecture Summary

### Intent

Provide a **highly available, horizontally scalable hosting platform** for a stateless customer-facing web application serving UK e-commerce traffic, using familiar AWS building blocks operable by a small engineering team.

### High-level shape

- **Single AWS Region:** `eu-west-2` (London) — UK customer latency and data residency alignment pending legal confirmation ([ASM-B1](04-assumptions.md)).
- **Multi-AZ** application tier: EC2 instances behind an Application Load Balancer across **at least two Availability Zones**.
- **DNS** via Route 53 to the load balancer.
- **Auto Scaling** to absorb seasonal peaks without year-round peak provisioning ([NFR-C2](03-non-functional-requirements.md), [CON-T4](05-constraints.md)).
- **Monitoring and alerting** via CloudWatch.
- **No RDS in v1** — relational data layer deferred; catalogue, orders, and inventory assumed to remain in **existing external systems** ([ASM-I1](04-assumptions.md)).
- **No WAF in v1** — deferred with documented accepted risk; revisit before first major sale if threat exposure warrants.

### Supporting context (not primary decision)

A **VPC** with public subnets (ALB) and private subnets (EC2) is required for this design but is treated as foundational networking — not a separate architectural debate in v1. NAT Gateway for private subnet egress to external APIs is **assumed** and **cost-significant**; exact subnet layout is deferred to implementation design.

### Discovery alignment

| Discovery input | v1 response |
|-----------------|-------------|
| Stateless application tier ([CON-T1](05-constraints.md)) | EC2 instances interchangeable; no sticky session requirement at load balancer if session externalised |
| High availability ([CON-T3](05-constraints.md)) | Multi-AZ ALB + ASG minimum two instances across AZs |
| Budget-conscious ([CON-F1](05-constraints.md)) | Small instance types; scale-out only under load; no WAF/RDS/multi-region in v1 |
| Small team ([CON-O1](05-constraints.md)) | EC2 + ASG — widely understood; avoids container platform learning curve in migration window |
| Seasonal peaks ([ASM-T2](04-assumptions.md)) | ASG scale policies; sizing validated by load test before cutover ([SC-T2](06-success-criteria.md)) |
| Session/basket continuity ([FR-2](02-functional-requirements.md)) | **Deferred** — session store not in v1 scope until stateless verification complete |

---

## 2. Proposed AWS Components

### Route 53

| Aspect | Detail |
|--------|--------|
| **Purpose** | Authoritative DNS for storefront domain; route customer traffic to the Application Load Balancer |
| **Why included** | Required for custom domain, cutover control, and health-checked routing if enabled. Supports low-downtime migration via DNS TTL reduction during cutover ([CON-P2](05-constraints.md)) |
| **Alternatives considered** | **External DNS registrar only** — rejected; AWS-native integration with ALB simplifies cutover and health routing. **CloudFront as DNS target** — deferred; adds CDN complexity not justified at v1 traffic levels |
| **Trade-offs** | Route 53 is low operational burden but **DNS propagation and TTL** affect rollback speed during cutover. Health-check failover helps AZ/ALB failure but is not instant |
| **Failure considerations** | Route 53 regional service failure is rare; mitigation is AWS-wide incident, not team-controlled. Misconfigured records cause full outage — change control required |
| **Cost considerations** | Low — hosted zone + query charges negligible at 5k daily users. Cost of **health checks** if used (~$0.50/health check/month) — optional in v1 |
| **Security considerations** | DNS hijack risk managed via registrar lock, IAM least privilege on hosted zone changes, MFA on account |

---

### Application Load Balancer (ALB)

| Aspect | Detail |
|--------|--------|
| **Purpose** | HTTPS termination, distribute HTTP(S) traffic across healthy EC2 instances in multiple AZs, expose health check endpoint for ASG |
| **Why included** | Native multi-AZ load balancing; HTTP-aware routing; integrates with ASG instance health; satisfies FR-9 health signalling at infrastructure layer |
| **Alternatives considered** | **Network Load Balancer** — rejected for v1; L4 only, less suited to path-based routing and standard web health checks unless TLS on instances. **Single EC2 with Elastic IP** — rejected; violates HA and CON-T3. **API Gateway** — rejected; not primary pattern for monolithic web storefront |
| **Trade-offs** | ALB has hourly cost regardless of traffic. Cross-AZ traffic to EC2 incurs **data transfer charges**. Simpler than NLB + separate TLS termination on instances |
| **Failure considerations** | ALB is managed multi-AZ. Failure shifts to **unhealthy backend pool** — if all instances fail health checks, site is down. Target connection errors during deploys if health check grace too short |
| **Cost considerations** | Fixed hourly component + LCU usage. Material at low traffic but justified for HA. Largest variable is cross-AZ data transfer during peak |
| **Security considerations** | TLS via **ACM** certificate; security group allows 443 from internet to ALB only; HTTP→HTTPS redirect. ALB access logs optional — deferred to control log cost ([NFR-S1](03-non-functional-requirements.md)) |

---

### Auto Scaling Group (ASG)

| Aspect | Detail |
|--------|--------|
| **Purpose** | Maintain minimum healthy instance count across AZs; add capacity during peak load; replace unhealthy instances automatically |
| **Why included** | Delivers CON-T3 HA (instance replacement), CON-T4 peak elasticity ([NFR-C2](03-non-functional-requirements.md)), and NFR-R3 safe rolling deploys when combined with appropriate policies |
| **Alternatives considered** | **Fixed EC2 pair (manual)** — rejected; no automatic recovery or peak scale. **ECS/EKS with Fargate/EC2** — viable but adds orchestration learning curve ([CON-O2](05-constraints.md)); may be v2 if team adopts containers. **Lambda** — rejected; monolith web app assumption ([ASM-A2](04-assumptions.md)) |
| **Trade-offs** | Scale-out is not instant — **warm-up time** during sudden spike may lag ([ASM-T2](04-assumptions.md) risk). Aggressive scaling increases cost; conservative scaling risks latency under peak |
| **Failure considerations** | ASG cannot scale if AMI boot fails, user data errors, or capacity insufficient in AZ. **Az imbalance** if one AZ impaired — spread across subnets required. Scaling policy on wrong metric causes late scale ([engineering principles — scalability](../../engineering-principles/scalability.md)) |
| **Cost considerations** | Pay per EC2 hour scaled. **Proposed starting bounds (assumption):** min **2**, desired **2**, max **4–6** pending load test. Max must be finance-approved before first sale |
| **Security considerations** | Launch template uses IAM instance profile — no long-lived keys on instances ([NFR-S2](03-non-functional-requirements.md)). Instance profile scoped to required AWS API calls only |

**Proposed scaling signals (subject to validation):**

- Scale out: average CPU > 70% for 5 minutes **or** ALB request count per target (preferred after baseline)
- Scale in: conservative cooldown to avoid flapping during sale

---

### EC2 Instances (across ≥2 Availability Zones)

| Aspect | Detail |
|--------|--------|
| **Purpose** | Run customer-facing web application; stateless compute tier serving HTTP from application server (e.g., nginx + app runtime) |
| **Why included** | Direct migration path from traditional hosting; team operability ([CON-O1](05-constraints.md)); sufficient for estimated 50–150 normal / 250–1,500 peak concurrent users ([ASM-T3](04-assumptions.md)) on appropriately sized instances |
| **Alternatives considered** | **Larger single instance** — rejected; SPOF. **Graviton (ARM)** — possible cost saving if application stack supports; validate before commit. **Reserved Instances / Savings Plans** — financial optimisation after steady-state, not day one |
| **Trade-offs** | EC2 requires patching, AMI management, and deploy automation — operational labour ([NFR-O1](03-non-functional-requirements.md)). Cheaper and simpler than container platforms at this scale **if** team knows EC2 |
| **Failure considerations** | Instance failure: ASG replaces. **Application bug** on all instances: ALB routes to all unhealthy — need rollback ([NFR-R3](03-non-functional-requirements.md)). **Dependency timeout** to external APIs can exhaust threads — circuit breakers are application concern |
| **Cost considerations** | **Proposed instance type (assumption):** `t3.small` or `t3.medium` — validate with load test. Burst credits on t3 may exhaust under sustained peak — monitor `CPUCreditBalance`. NAT Gateway cost often exceeds EC2 at low scale — flag in cost review |
| **Security considerations** | Instances in **private subnets**; no SSH from internet — SSM Session Manager preferred. Security group denies direct inbound from internet. Patch cadence documented |

**Distribution:** ASG spans **eu-west-2a** and **eu-west-2b** (minimum). Third AZ optional — deferred to reduce complexity in v1 unless tier confirmation demands it.

---

### Security Groups

| Aspect | Detail |
|--------|--------|
| **Purpose** | Stateful firewall controlling traffic between internet, ALB, and EC2 tiers |
| **Why included** | Baseline network segmentation ([NFR-S2](03-non-functional-requirements.md)); minimum blast radius between public edge and application |
| **Alternatives considered** | **NACLs as primary control** — rejected; coarse, error-prone. NACLs may supplement for defence in depth — deferred. **Single flat security group** — rejected; violates least exposure |
| **Trade-offs** | Simple two-tier model (ALB SG, App SG) vs micro-segmentation — v1 chooses simplicity |
| **Failure considerations** | Misconfigured SG causes **total connectivity loss** — common migration mistake. Changes are immediate — no rollback mechanism beyond revert |
| **Cost considerations** | No direct charge |
| **Security considerations** | **Proposed rules (conceptual):** ALB SG — inbound 443 from `0.0.0.0/0` (or geo-restriction deferred); outbound to App SG on app port. App SG — inbound from ALB SG only; outbound 443 to internet for external APIs (via NAT) and integration endpoints |

---

### CloudWatch

| Aspect | Detail |
|--------|--------|
| **Purpose** | Metrics, logs, and alarms for infrastructure and application health; support NFR-O2 detectability |
| **Why included** | Default integration with ALB, ASG, EC2; operable by small team without third-party APM cost in v1 |
| **Alternatives considered** | **Third-party APM (Datadog, etc.)** — valuable but adds cost and integration ([CON-F1](05-constraints.md)); defer unless CloudWatch insufficient after launch. **Enhanced monitoring everywhere** — rejected; cost vs value |
| **Trade-offs** | CloudWatch Logs retention costs accumulate — set retention explicitly ([NFR-OB1](03-non-functional-requirements.md)). Limited distributed tracing unless X-Ray added — **deferred** |
| **Failure considerations** | Alarm fatigue if thresholds too tight ([observability principles](../../engineering-principles/observability.md)). During AWS API impairment, console access may degrade — runbooks should not depend solely on console |
| **Cost considerations** | Log ingestion and retention primary variable. Start with **14–30 day retention** for application logs; tune after observing volume |
| **Security considerations** | No PII in logs ([NFR-S1](03-non-functional-requirements.md)). IAM restricts alarm and log access. Log group encryption at rest — enable if handling sensitive debug output |

**Proposed minimum alarms (v1):**

| Alarm | Source | Rationale |
|-------|--------|-----------|
| Unhealthy host count ≥ 1 | ALB target group | Instance or deploy failure |
| 5xx rate above threshold | ALB | Customer-visible errors ([NFR-OB1](03-non-functional-requirements.md)) |
| ASG at max capacity | ASG | Peak headroom exhausted — sale risk |
| CPU sustained high | EC2 / ASG | Scale policy failure or undersizing |

Application-level checkout failure metrics require **application instrumentation** — not CloudWatch alone. Flagged as gap for implementation phase.

---

### WAF — **Deferred (not in v1)**

| Aspect | Detail |
|--------|--------|
| **Purpose** | Web application firewall — filter OWASP Top 10, bots, rate-based rules at edge |
| **Why not included in v1** | Budget-conscious constraint ([CON-F1](05-constraints.md)); small team lacks WAF tuning capacity; baseline traffic (~5k daily users) does not alone justify managed WAF cost before validated threat model. ALB + security groups + TLS provide baseline |
| **Alternatives considered** | **AWS WAF on ALB** — recommended **before first major sale** if public attack surface or bot traffic observed. **CloudFront + WAF** — stronger edge position; deferred with CDN decision |
| **Trade-offs** | Deferring accepts **application-layer attack risk** (SQLi, XSS mitigated primarily in app code). Document as **P1 accepted risk** pending sale calendar |
| **Failure considerations** | Misconfigured WAF rules block legitimate customers — common operational risk when enabled without care |
| **Cost considerations** | WAF ACL hourly charge + rule + request fees — non-trivial at low margin e-commerce |
| **Security considerations** | If payment redirect model keeps card data off origin ([NFR-S3](03-non-functional-requirements.md)), WAF priority is storefront abuse and availability attacks, not PCI boundary alone |

**Revisit trigger:** 60 days before first major sale, or earlier if vulnerability scan or incident warrants.

---

### RDS — **Deferred (not in v1)**

| Aspect | Detail |
|--------|--------|
| **Purpose** | Managed relational database for application persistence |
| **Why not included in v1** | Discovery indicates **stateless application tier** ([CON-T1](05-constraints.md)) with catalogue, inventory, orders, and payment likely in **existing external systems** ([ASM-I1](04-assumptions.md)). No discovery requirement mandates migrating relational data to AWS in this phase. Introducing RDS adds multi-AZ cost, backup/restore operations, patching, and connection pool management — disproportionate if web tier is primarily an **integration and presentation layer** |
| **Alternatives considered** | **RDS Multi-AZ** — required if session/basket or local catalogue cache moves to SQL; evaluate after ASM-A1 validation. **ElastiCache (Redis)** — lighter option if only session/basket externalisation needed ([FR-2](02-functional-requirements.md)). **DynamoDB** — viable for session tokens; deferred. **Keep session in existing backend** — preferred if already operational |
| **Trade-offs** | Deferring RDS simplifies v1 but **blocks** architecture if discovery reveals hidden local database or mandatory session store on AWS |
| **Failure considerations** | If session store is later required without design, sticky sessions on ALB become tempting — **anti-pattern** for CON-T1; must externalise state instead |
| **Cost considerations** | RDS Multi-AZ roughly **2×** single-instance database cost plus storage — material for budget-conscious org |
| **Security considerations** | If RDS added later: private subnets, encryption at rest (KMS), no public accessibility, least-privilege DB credentials via Secrets Manager |

**Revisit trigger:** Application architecture review confirms session/basket storage location and whether any datastore migrates with storefront.

---

## 3. Request Flow

Narrative only — no diagram in v1.

1. **Customer (UK)** resolves storefront domain via **Route 53** → ALB DNS name.
2. **HTTPS request** arrives at **Application Load Balancer** (TLS terminated with ACM certificate).
3. ALB selects healthy target in **Auto Scaling Group** (instance in AZ-a or AZ-b).
4. **EC2 instance** (private subnet) processes request:
   - Serves static/cacheable content from local disk or app memory where appropriate.
   - Calls **external APIs** (catalogue, inventory, payment, orders) over HTTPS via **NAT Gateway** — integrations remain out of scope for v1 infrastructure.
5. **Response** returns through ALB to customer.
6. **Payment redirect** (if applicable): customer may leave origin to payment provider — return URL hits same flow.
7. **Metrics and logs** emitted to **CloudWatch**; alarms notify on-call per coverage model ([NFR-O3](03-non-functional-requirements.md)).

**Session/basket note:** If session state lives in external store, EC2 instances remain interchangeable and ALB need not use stickiness. If not yet externalised, **migration blocker** — see [Deferred Decisions](#9-deferred-decisions).

---

## 4. Availability Design

### Target (proposed, unvalidated)

Align with **99.9% monthly** ([NFR-A1](03-non-functional-requirements.md)) for storefront — stricter sale-window target pending business confirmation.

### Mechanisms in v1

| Mechanism | Contribution |
|-----------|--------------|
| Multi-AZ ALB | Survives single AZ impairment for load balancer |
| ASG min 2 across ≥2 AZs | Survives single instance failure; partial AZ impairment maintains capacity |
| ALB health checks | Removes unhealthy instances from rotation |
| ASG automatic replacement | Recovers from instance failure without manual intervention |
| Rolling deployments | With correct health check grace, reduces deploy-induced outage ([NFR-R3](03-non-functional-requirements.md)) |

### What v1 does **not** provide

- **Multi-region DR** — deferred; RTO/RPO for regional failure not met ([NFR-R1](03-non-functional-requirements.md) regional scenario).
- **Zero-downtime during AZ-wide impairment** — if all instances in both AZs fail simultaneously, outage occurs until ASG recovers.
- **Dependency HA** — external payment or inventory API outage is **not** solved by this design ([discovery risk — integration fragility](07-architecture-discovery-summary.md)).

### Graceful degradation

Application must implement degraded modes ([NFR-A2](03-non-functional-requirements.md)) — e.g., browse without checkout if payment API unavailable. **Not guaranteed by infrastructure alone.**

---

## 5. Scalability Design

### Baseline assumption

~5,000 daily users; peak concurrent **250–1,500** if 5–10× multiplier holds ([ASM-T2](04-assumptions.md), [ASM-T3](04-assumptions.md)) — **unvalidated**.

### v1 approach

- **Horizontal scale** via ASG add instances — aligns with stateless tier ([CON-T1](05-constraints.md)).
- **Scale for next order of magnitude**, not decade — max instance count capped until load test proves need ([scalability principle](../../engineering-principles/scalability.md)).
- **No CDN in v1** — may add CloudFront if static asset load or latency drives it; UK-only audience reduces edge urgency.

### Bottleneck watchlist

| Potential bottleneck | v1 mitigation | Gap |
|---------------------|---------------|-----|
| EC2 CPU / threads | ASG scale out | Sudden spike lag |
| External API rate limits | App timeouts, circuit breakers | Not infrastructure |
| NAT Gateway bandwidth | Monitor; split AZ NAT cost | May need VPC endpoints later |
| Database (external) | Out of AWS scope | Partner scaling |

**Load test required** before cutover ([SC-T2](06-success-criteria.md)).

---

## 6. Security Design

### Network

- VPC with public subnets (ALB only) and private subnets (EC2).
- No inbound internet access to EC2.
- Egress via NAT for external API calls — restrict outbound SG where integration endpoints are known ([CON-T2](05-constraints.md)).

### Identity and access

- IAM roles for EC2 (instance profile) — no access keys on instances.
- Human access via SSO + IAM role for engineers; SSM Session Manager for instance access.
- Break-glass procedure — documented, not embedded in v1 proposal.

### Data protection

- TLS 1.2+ in transit (ALB + ACM).
- UK GDPR: personal data in logs redacted; subprocessors documented ([NFR-S1](03-non-functional-requirements.md)).
- Payment card data **not** stored on origin if redirect/hosted payment model confirmed ([NFR-S3](03-non-functional-requirements.md)) — **validation required**.

### Threat surface

- Public HTTPS endpoint — brute force, scraping, application exploits.
- WAF deferred — compensating controls: secure coding, dependency patching, rate limiting in application where feasible.

---

## 7. Observability Design

### SLIs (proposed minimum)

| SLI | Source (v1) | Gap |
|-----|--------------|-----|
| Availability (synthetic) | Route 53 health check or external synthetic — optional | May add canary in v1.1 |
| ALB 5xx rate | CloudWatch | Symptom, not journey-level |
| Target health | ALB metrics | Infrastructure only |
| Latency p95 | ALB `TargetResponseTime` | Approximate; not full user RUM |
| Checkout success rate | **Application metrics** | **Not in infrastructure v1 — required before sale** |

### Logging

- ALB access logs — optional initially (cost); enable if debug needed.
- Application structured logs to CloudWatch Logs — JSON with request ID; no PII.

### Alerting

- Symptom-based alarms tied to runbooks ([NFR-O2](03-non-functional-requirements.md)).
- On-call coverage must match alert design ([CON-O3](05-constraints.md)) — paging without responder is harmful.

---

## 8. Cost Considerations

**Order-of-magnitude only — not a quote.** Budget envelope undefined ([ASM-F1](04-assumptions.md)).

### Expected cost drivers (normal month)

| Component | Cost profile |
|-----------|--------------|
| EC2 (2 × t3.small/medium) | Largest predictable compute line |
| ALB | Fixed hourly + LCU |
| NAT Gateway | **Often underestimated** — hourly + per-GB processed |
| Route 53 | Low |
| CloudWatch | Moderate if verbose logging |
| Data transfer | Cross-AZ (ALB→EC2), egress to internet |

### Peak month

- ASG scaled to max — EC2 hours increase linearly.
- NAT and data transfer may spike with traffic.
- **Pre-event forecast required** ([SC-F2](06-success-criteria.md), [NFR-C1](03-non-functional-requirements.md)).

### Cost controls

- Resource tagging (environment, service, cost centre).
- AWS Budgets with alert at 80% of approved envelope — once defined.
- Scale-in policies to avoid overnight peak capacity after sale.
- Revisit NAT topology (single vs per-AZ NAT) — cost vs AZ resilience trade-off.

### Deliberately excluded from v1 cost

- WAF, RDS, CloudFront, X-Ray, third-party APM.

---

## 9. Deferred Decisions

| Decision | Reason deferred | Revisit trigger |
|----------|-----------------|-----------------|
| Session/basket store (ElastiCache, RDS, existing API) | ASM-A1 unverified; FR-2 requires behaviour not technology | Application architecture review |
| WAF | Cost, ops complexity; threat model incomplete | 60 days before major sale |
| RDS / any AWS datastore | External systems assumed for commerce data | Integration inventory complete |
| CloudFront / CDN | Latency may be acceptable from eu-west-2 for UK | Load test p95 fails NFR-P1 |
| Multi-region DR | Cost and complexity exceed v1 scope | Business defines regional RTO |
| VPC endpoints | NAT cost vs endpoint cost trade-off | Sustained high NAT charges |
| Container migration (ECS) | EC2 sufficient for v1; team maturity ([CON-O2](05-constraints.md)) | Operational pain with AMI/deploy |
| Exact instance type and ASG max | Requires load test data | SC-T2 completion |
| Route 53 health-checked failover | May duplicate ALB AZ resilience | Cutover planning |
| X-Ray / distributed tracing | Team capacity | Checkout latency incidents |

---

## 10. Known Risks

| Risk | Severity | v1 mitigation | Residual |
|------|----------|---------------|----------|
| Peak load underestimated | High | ASG max + load test | May still fail if multiplier > assumed |
| Hidden local state on instances | High | Discovery validation ASM-A1 | Migration scope explosion if true |
| External API outage | High | App degradation patterns | Customer impact unchanged |
| Budget vs HA mismatch | High | Simple design, no WAF/RDS | May not meet executive HA interpretation |
| NAT as cost and SPoF | Medium | Monitor; design review NAT | AZ NAT failure affects egress |
| No WAF | Medium | App security hygiene | Elevated exploit exposure |
| CloudWatch-only observability | Medium | ALB metrics | Checkout SLI gap until app metrics |
| DNS cutover error | Medium | Low TTL rehearsal | Rollback depends on TTL |
| t3 CPU credit exhaustion | Medium | Monitor credits; consider t3 unlimited or m-family | Latency under sustained peak |
| Single-region | Medium | Accepted for v1 | Regional AWS impairment = outage |

---

## 11. Review Questions

Questions for architecture review meeting — map to [P0/P1/P2 framework](../../architecture-review-framework/README.md).

### P0 candidates (must resolve before build approval)

1. Is application tier **confirmed stateless** — where does session/basket live today?
2. What is **approved ASG max size** and monthly budget range ([ASM-F1](04-assumptions.md))?
3. Are **all external integrations** reachable from private subnet via NAT — IP allowlisting implications?
4. What is **acceptable outage duration** during named sale ([ASM-B3](04-assumptions.md))?
5. Is **payment redirect** confirmed — card data never on EC2?

### P1 candidates (significant risk)

6. Is **WAF deferral** accepted until pre-sale review?
7. Who is **on-call** during first sale — does coverage match alerting ([CON-O3](05-constraints.md))?
8. What **load test scenario** defines pass/fail for SC-T2?
9. Is **NAT single-AZ** acceptable vs per-AZ NAT cost for resilience?
10. What is **rollback plan** if cutover fails — DNS revert time?

### P2 candidates (improvements)

11. Graviton vs x86 for cost?
12. CloudFront for static assets?
13. Route 53 health checks vs synthetic external monitor?
14. Reserved capacity after 90-day steady state?

---

## Assumptions Carried from Discovery

This proposal **depends on** the following remaining true — see [04-assumptions.md](04-assumptions.md):

- ASM-A1: Stateless application tier (session externalised)
- ASM-I1: Catalogue, orders, payment remain external APIs
- ASM-T2/T3: Peak multiplier and concurrent estimates (for sizing only)
- ASM-B1: UK-primary, eu-west-2 acceptable for data residency
- ASM-O1: Team can operate EC2 + ASG + ALB

If any assumption fails, **this proposal requires revision** (v2).

---

## Recommended Next Steps

1. Architecture review with P0/P1 disposition recorded.
2. ADR-0002 (or next sequence): accept v1 proposal or document deltas.
3. Application session/state verification — unblock or add session store.
4. Integration inventory — NAT, SG egress, partner allowlisting.
5. Load test plan tied to ASM-T3 validated numbers.
6. Cost estimate spreadsheet for finance — normal vs peak month.
7. Diagram v1 after review feedback — not before.

---

## Related Documents

- [07 — Architecture Discovery Summary](07-architecture-discovery-summary.md)
- [03 — Non-functional Requirements](03-non-functional-requirements.md)
- [05 — Constraints](05-constraints.md)
- [06 — Success Criteria](06-success-criteria.md)
- [Architecture Review Framework](../../architecture-review-framework/README.md)
- [Decision Matrix](../../decision-matrix.md)

---

*v1 is a starting point for review. It is expected to change.*
