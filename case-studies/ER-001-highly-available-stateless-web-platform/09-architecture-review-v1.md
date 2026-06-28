# ER-001: Architecture Review v1

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 09 — Architecture Review v1  
**Reviewed artifact:** [08-architecture-proposal-v1.md](08-architecture-proposal-v1.md)  
**Reviewer role:** Principal Cloud Architect  
**Review date:** 2026-06-28  
**Status:** Complete — disposition open

---

## Review Purpose

Formal architecture review of proposal v1 against discovery documents (01–07), [engineering principles](../../engineering-principles/README.md), [architecture review framework](../../architecture-review-framework/README.md), [architecture tiers](../../architecture-tiers.md), [organisational constraints](../../organisational-constraints.md), [principle conflicts](../../principle-conflicts.md), and [decision matrix](../../decision-matrix.md) thinking.

This review **does not rewrite the architecture**. It records findings, severity, and conditions for progression to ADR and build.

---

## Review Summary

| Severity | Count |
|----------|-------|
| P0 — Launch Blocker | 6 |
| P1 — Significant Risk | 12 |
| P2 — Improvement Opportunity | 8 |

**Overall disposition:** The proposal direction — multi-AZ ALB + ASG + EC2 in `eu-west-2` — is **reasonable** for a small team, budget-conscious, stateless storefront **if** critical assumptions hold. The proposal is **honest about gaps** but several gaps are **P0 for ADR and build**, not polish.

**Approval status:** **Not approved yet**

---

## 1. Business Alignment

### FIND-BA-01

| Field | Detail |
|-------|--------|
| **Severity** | P0 |
| **Finding** | Business has not quantified acceptable outage duration during named seasonal sales ([ASM-B3](04-assumptions.md)). Proposal targets 99.9% monthly ([NFR-A1](03-non-functional-requirements.md)) while discovery proposed Tier 1 checkout / stricter sale-window policy — **misaligned and unvalidated**. |
| **Why it matters** | Without a numeric sale-window tolerance, the team cannot justify multi-AZ cost, ASG max, WAF timing, or on-call investment. Executive HA expectation may exceed what v1 delivers ([principle conflict: reliability vs cost](../../principle-conflicts.md)). |
| **Recommended action** | Commercial sponsor to document maximum acceptable **complete storefront outage** during live sale (minutes) and sign tier assignment for checkout path. |
| **Owner** | Commercial / Product Sponsor |
| **Status** | open |

### FIND-BA-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | Proposal addresses infrastructure HA but not **dependency HA** — catalogue, payment, inventory APIs remain external ([ASM-I1](04-assumptions.md)). Business problem ([01-business-problem.md](01-business-problem.md)) cites peak-period risk; migration may not improve perceived availability if dependencies fail. |
| **Why it matters** | SC-B1 and SC-B2 success criteria can fail despite healthy AWS infrastructure. Marketing confidence ([SC-B2](06-success-criteria.md)) requires end-to-end journey reliability. |
| **Recommended action** | Integration inventory with partner SLAs; document degraded-mode UX (NFR-A2) as business-approved behaviour when payment/inventory unavailable. |
| **Owner** | Engineering Lead + Commercial Sponsor |
| **Status** | open |

### FIND-BA-03

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | Migration timeline relative to next major sale ([ASM-O4](04-assumptions.md), [CON-P1](05-constraints.md)) not referenced in proposal v1. |
| **Why it matters** | Scope cut decisions (WAF, load test depth, dual-run duration) depend on deadline. |
| **Recommended action** | Record hard migration deadline and named sale date in ER-001 programme section. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 2. Requirements Traceability

### FIND-RT-01

| Field | Detail |
|-------|--------|
| **Severity** | P0 |
| **Finding** | **FR-2 (session/basket continuity)** is not architecturally supported in v1 — explicitly deferred pending ASM-A1 validation. Proposal states migration blocker if session not externalised; **no verification completed**. |
| **Why it matters** | Must-have functional requirement without design backing. Violates [engineering workflow](../../engineering-workflow.md) gate — architecture proposal before session model confirmed. |
| **Recommended action** | Application architecture review: document session/basket storage today; add session store to v1.1 proposal **or** confirm external store + update FR traceability matrix. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-RT-02

| Field | Detail |
|-------|--------|
| **Severity** | P0 |
| **Finding** | **FR-5 / NFR-S3 (payment boundary)** — proposal assumes redirect/hosted payment; **not validated**. Card data on EC2 would change security scope materially. |
| **Why it matters** | Compliance, logging, WAF priority, and SG egress rules depend on payment model. Discovery flagged as critical gap ([07-architecture-discovery-summary.md](07-architecture-discovery-summary.md)). |
| **Recommended action** | Document payment flow with provider; security review sign-off that origin never stores/processes PAN. |
| **Owner** | Engineering Lead + Security |
| **Status** | open |

### FIND-RT-03

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **NFR-OB1 / checkout success SLI** — proposal acknowledges gap; CloudWatch-only cannot measure checkout journey. SC-T2 and NFR-O2 require application metrics **before sale** — not mapped to proposal components. |
| **Why it matters** | [Observability principle](../../engineering-principles/observability.md): symptom-based alerting on checkout failure is Tier 1 expectation ([architecture tiers](../../architecture-tiers.md)). |
| **Recommended action** | Add implementation-phase requirement: emit checkout/basket error metrics and alarms before production cutover; trace to FR-4, FR-5. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-RT-04

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **NFR-A2 (graceful degradation)** — required at discovery; proposal delegates entirely to application with no ranked essential journey list validated by business. |
| **Why it matters** | Cannot design circuit breakers or partial outage comms without FR priority ranking from discovery ([03-non-functional-requirements.md](03-non-functional-requirements.md) validation required). |
| **Recommended action** | Stakeholder sign-off: ordered degradation (browse vs checkout vs payment). |
| **Owner** | Commercial Sponsor |
| **Status** | open |

### FIND-RT-05

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | **FR-8 (campaign landing pages)** marked should-have — proposal does not discuss CMS/static content serving. |
| **Why it matters** | Seasonal campaigns drive peak traffic ([01-business-problem.md](01-business-problem.md)). May affect CloudFront deferral. |
| **Recommended action** | Confirm CMS integration path; note in v1.1 if static asset CDN needed. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 3. Availability and Resilience

### FIND-AR-01

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | Proposal aligns with **Tier 2** availability (99.9%) at infrastructure layer but discovery proposed **Tier 1 for checkout** ([03-non-functional-requirements.md](03-non-functional-requirements.md)). Tier 1 expects synthetic canaries, 24/7 paging on SLO breach, tested DR — **not in v1**. |
| **Why it matters** | [Architecture tiers](../../architecture-tiers.md) + [organisational constraints](../../organisational-constraints.md): claiming Tier 1 checkout with Tier 2 ops is a **principle conflict** ([reliability vs cost](../../principle-conflicts.md), on-call vs tier). |
| **Recommended action** | Formal tier sign-off: accept Tier 2 for entire storefront **or** add Tier 1 checkout controls (canary, on-call, stricter RTO evidence). |
| **Owner** | Engineering Lead + Commercial Sponsor |
| **Status** | open |

### FIND-AR-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **No DR design** for regional failure — proposal correctly excludes multi-region but does not document RTO/RPO for regional AWS impairment vs business expectation ([NFR-R1](03-non-functional-requirements.md)). |
| **Why it matters** | Single-region is accepted risk only if business accepts regional RTO measured in hours/days. |
| **Recommended action** | Document accepted regional RTO/RPO as explicit risk with owner; include in ADR consequences. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-AR-03

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **Backup/restore and RPO** for any AWS-resident data (logs, AMIs, session store if added) not defined. RDS deferred — acceptable — but recovery story for **rebuild EC2 from IaC** not stated as tested requirement ([SC-T3](06-success-criteria.md)). |
| **Why it matters** | [Reliability principle](../../engineering-principles/reliability.md): backups without restore testing are insufficient. |
| **Recommended action** | Define minimum recovery path: AMI/IaC rebuild + external API reconnection; schedule game day before cutover. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-AR-04

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | Rolling deployment and health check grace period mentioned but **not specified** — bad deploy risk ([NFR-R3](03-non-functional-requirements.md)). |
| **Why it matters** | Self-inflicted outage during sale is explicit failure criterion ([06-success-criteria.md](06-success-criteria.md)). |
| **Recommended action** | Implementation standard: min healthy percent, grace period, rollback runbook — defer detail to implementation with trigger before first prod deploy. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 4. Security

### FIND-SE-01

| Field | Detail |
|-------|--------|
| **Severity** | P0 |
| **Finding** | **Threat model not produced** — [engineering workflow](../../engineering-workflow.md) lists threat model stage; proposal defers WAF without structured threat analysis (assets, actors, attack surfaces). |
| **Why it matters** | [Security by design](../../engineering-principles/security-by-design.md) requires threat-informed controls. WAF deferral may be correct but must be **accepted residual risk**, not absence of analysis. |
| **Recommended action** | Lightweight threat model document: public storefront, admin access, payment redirect, external APIs; drives WAF decision and SG egress rules. |
| **Owner** | Security / Engineering Lead |
| **Status** | open |

### FIND-SE-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **WAF deferred** — proposal documents P1 accepted risk. Public e-commerce during adversarial sale periods (bots, scraping) without rate limiting at edge. |
| **Why it matters** | Availability attacks and application exploit attempts are realistic during high-traffic events. [Security vs simplicity](../../principle-conflicts.md): deferral acceptable only with documented trigger and owner. |
| **Recommended action** | Accept deferral until 60 days pre-sale **with** application rate limiting minimum; calendar WAF review milestone. |
| **Owner** | Security + Engineering Lead |
| **Status** | open |

### FIND-SE-03

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **ALB access logs optional** — may hinder incident investigation and UK GDPR audit trail for access to personal data in URLs. |
| **Why it matters** | NFR-S1 auditability; cost vs security trade-off not decided. |
| **Recommended action** | Enable ALB logs with short retention (e.g., 7–14 days) or S3 with lifecycle; redact query params if needed. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-SE-04

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **Break-glass and human access** — mentioned conceptually; separation of duties, MFA, IAM review process not defined ([architecture review framework — IAM](../../architecture-review-framework/README.md)). |
| **Why it matters** | Small team risk: shared credentials undermine entire design. |
| **Recommended action** | Document IAM model for prod access (SSO roles, no long-lived keys); break-glass procedure. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-SE-05

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | **Geo-restriction** on ALB SG deferred — UK-only assumption ([ASM-B1](04-assumptions.md)) not enforced at edge. |
| **Why it matters** | Reduces attack surface; low effort if truly UK-only. |
| **Recommended action** | Evaluate AWS WAF geo match or CloudFront geo restriction if WAF adopted. |
| **Owner** | Security |
| **Status** | open |

---

## 5. Networking

### FIND-NW-01

| Field | Detail |
|-------|--------|
| **Severity** | P0 |
| **Finding** | **Integration reachability not validated** — proposal assumes NAT egress to external APIs ([CON-T2](05-constraints.md)). Partner IP allowlisting, fixed egress IPs, VPN, or private link requirements **unknown**. |
| **Why it matters** | Common migration failure: app works in old host, fails from AWS egress IP. Blocks cutover ([07-architecture-discovery-summary.md](07-architecture-discovery-summary.md) critical gap). |
| **Recommended action** | Integration inventory: endpoint, allowlisting, timeout, failure mode; NAT Gateway with Elastic IP if partners require it. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-NW-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **NAT Gateway topology undecided** — single NAT vs per-AZ NAT. Proposal flags cost vs AZ resilience trade-off without decision. |
| **Why it matters** | Single NAT = AZ failure loses egress → checkout fails even if EC2 healthy ([principle conflict: reliability vs cost](../../principle-conflicts.md)). NAT often **largest cost line** at low traffic ([cost awareness](../../engineering-principles/cost-awareness.md)). |
| **Recommended action** | Decision matrix entry: single NAT (cost) vs dual NAT (resilience); finance + engineering sign-off. |
| **Owner** | Engineering Lead + Finance |
| **Status** | open |

### FIND-NW-03

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **VPC/subnet design deferred** — acceptable for v1 proposal but CIDR, subnet sizing, and AZ subnet mapping needed before build. |
| **Why it matters** | Late VPC changes are painful; cross-AZ data transfer costs depend on layout. |
| **Recommended action** | Implementation design doc: VPC CIDR, 2+ AZ public/private subnets, routing tables. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-NW-04

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | **VPC endpoints** deferred — SSM, CloudWatch Logs, EC2 API via NAT adds cost and NAT dependency for management plane. |
| **Why it matters** | Operational traffic through NAT during incident is fragile. |
| **Recommended action** | Add interface endpoints for SSM and CloudWatch Logs at implementation if NAT cost/SPoF confirmed. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 6. Cost

### FIND-CO-01

| Field | Detail |
|-------|--------|
| **Severity** | P0 |
| **Finding** | **No numeric budget envelope** ([ASM-F1](04-assumptions.md), [CON-F2](05-constraints.md)). Proposal provides order-of-magnitude drivers only. Finance cannot approve; ASG max unbounded commercially. |
| **Why it matters** | SC-F1, SC-F2, NFR-C1 require measurable targets. [Decision matrix](../../decision-matrix.md): cost criterion cannot be scored. |
| **Recommended action** | Finance-approved range: normal month, peak month, migration month; cap ASG max against peak forecast. |
| **Owner** | Finance + Engineering Lead |
| **Status** | open |

### FIND-CO-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **NAT Gateway cost risk** explicitly flagged in proposal but not quantified — may exceed EC2 at 5k daily users scale. |
| **Why it matters** | Budget-conscious org ([CON-F1](05-constraints.md)); surprise bill undermines programme. |
| **Recommended action** | AWS Pricing Calculator estimate: NAT + ALB + 2 EC2 normal vs 6 EC2 peak; compare to current hosting ([SC-F1](06-success-criteria.md)). |
| **Owner** | Engineering Lead + Finance |
| **Status** | open |

### FIND-CO-03

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **Dual-run migration cost** not estimated — parallel old/new environments during cutover ([07-architecture-discovery-summary.md](07-architecture-discovery-summary.md) medium risk). |
| **Why it matters** | CON-P2 low-downtime cutover implies overlap period. |
| **Recommended action** | Include dual-run weeks in migration month forecast. |
| **Owner** | Finance |
| **Status** | open |

### FIND-CO-04

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | Tagging strategy mentioned; **mandatory tag keys** not defined for allocation. |
| **Why it matters** | FinOps and SC-F1 attribution. |
| **Recommended action** | Define tags: Environment, Service, CostCentre, Owner before first resource created. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 7. Operations

### FIND-OP-01

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **On-call model undefined** ([NFR-O3](03-non-functional-requirements.md), [CON-O3](05-constraints.md)). Proposal alarms assume pager; discovery does not confirm 24/7 or sale-window coverage. |
| **Why it matters** | [Organisational constraints](../../organisational-constraints.md): paging without responder is harmful. Tier mismatch if Tier 1 alerts on business-hours team. |
| **Recommended action** | Document on-call roster for cutover week and first sale; align alarm severity to coverage hours. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-OP-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **Runbooks not referenced** — SC-O1 requires operability without consultants; no runbook list for ALB unhealthy, ASG max, NAT failure, DNS rollback. |
| **Why it matters** | [Operational excellence](../../engineering-principles/operational-excellence.md): 3am test fails if runbooks missing. |
| **Recommended action** | Runbook backlog: top 5 failure modes from proposal §10; link from CloudWatch alarms. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-OP-03

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **EC2 patching and AMI pipeline** — operational labour acknowledged but not planned ([NFR-O1](03-non-functional-requirements.md)). |
| **Why it matters** | Unpatched instances are security and reliability debt. |
| **Recommended action** | Patch cadence; golden AMI or config management approach before prod. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-OP-04

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | **CI/CD and IaC** referenced in success criteria (SC-T4) but absent from proposal v1. |
| **Why it matters** | Manual deploy undermines NFR-R3. |
| **Recommended action** | Implementation-phase ADR or design note for deploy pipeline — not blocking architectural direction. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 8. Observability

### FIND-OB-01

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **No synthetic/canary monitoring** for customer journey — optional Route 53 health check insufficient for Tier 1 checkout ([architecture tiers](../../architecture-tiers.md)). |
| **Why it matters** | ALB health only confirms instance liveness, not checkout path or dependency health. |
| **Recommended action** | Implement synthetic checkout canary (even external lightweight) before first sale; minimum for SC-O2. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-OB-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **SLO/error budget not formalised** — proposed 99.9% without error budget policy or release trade-off ([observability principle](../../engineering-principles/observability.md)). |
| **Why it matters** | Without SLO, alert thresholds are arbitrary. |
| **Recommended action** | Define SLI (availability via ALB 5xx + synthetic); SLO 99.9%; document error budget consumption during deploys. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-OB-03

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | **X-Ray/tracing deferred** — acceptable at v1 scale if team small; checkout latency debug harder across external APIs. |
| **Why it matters** | MTTR during sale incidents. |
| **Recommended action** | Structured logging with correlation ID across outbound integration calls as minimum. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 9. Scalability

### FIND-SC-01

| Field | Detail |
|-------|--------|
| **Severity** | P0 |
| **Finding** | **Peak load unvalidated** — ASG max 4–6 and instance type based on ASM-T2/T3 estimates only. **Load test pass/fail (SC-T2) not defined.** |
| **Why it matters** | Primary discovery risk ([07-architecture-discovery-summary.md](07-architecture-discovery-summary.md)). Architecture cannot be approved for sale cutover without validated scenario. |
| **Recommended action** | Obtain analytics; define load test profile (RPS, concurrent users, duration spike); pass criteria linked to NFR-P2. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-SC-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **t3 burst credit exhaustion** under sustained peak — proposal notes risk; no mitigation chosen (unlimited mode, m-family, scale-out earlier). |
| **Why it matters** | CPU credits exhaust silently → latency spike during sale ([scalability principle](../../engineering-principles/scalability.md)). |
| **Recommended action** | Load test monitors `CPUCreditBalance`; choose instance family before prod. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-SC-03

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **Scale-out lag** on sudden spike — CPU 5-minute average may be too slow for flash sale ([ASM-T2](04-assumptions.md)). |
| **Why it matters** | Auto-scaling on wrong metric is listed common mistake in principles. |
| **Recommended action** | Prefer ALB request count per target after baseline; pre-warm ASG before known sale events. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-SC-04

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | **CloudFront deferred** — may be fine for UK + eu-west-2; static asset load unknown. |
| **Why it matters** | NFR-P1 latency if assets served from EC2. |
| **Recommended action** | Load test measures static vs dynamic; CDN decision after data. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 10. Deferred Decisions

### FIND-DD-01

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **Session/basket store deferral** is appropriate for proposal stage but **cannot defer to implementation** if ASM-A1 fails — must gate ADR. |
| **Why it matters** | Deferred decision with revisit trigger is good practice; conflating "deferred" with "optional" is not. |
| **Recommended action** | Explicit programme gate: no ADR acceptance until session model documented. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-DD-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **RDS deferral justified** by ASM-I1 — decision matrix supports simplicity + cost win if web tier is integration layer. **Revisit trigger correct.** |
| **Why it matters** | If wrong, ElastiCache/RDS ADR required — acceptable deferred **with trigger**. |
| **Recommended action** | Confirm no local DB in application repo/deploy artifact. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-DD-03

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | Deferral register in proposal §9 is **well-structured** with revisit triggers — aligns with [engineering workflow](../../engineering-workflow.md) and playbook standards. |
| **Why it matters** | Positive finding — maintain in ADR. |
| **Recommended action** | Copy deferral table into ADR appendix when written. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 11. Assumptions

### FIND-AS-01

| Field | Detail |
|-------|--------|
| **Severity** | P0 |
| **Finding** | **ASM-A1 (stateless tier)** — entire proposal rests on this; status **unverified** ([04-assumptions.md](04-assumptions.md) critical priority). |
| **Why it matters** | Hidden local state invalidates ASG design and FR-2 approach. |
| **Recommended action** | Code and runtime audit; document outcome in ER-001 assumption register update. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-AS-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **ASM-I1 (external backends)** — if false, proposal scope and cost model wrong. |
| **Why it matters** | May require RDS, VPN, or migration wave 2. |
| **Recommended action** | Integration workshop; update assumption register. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-AS-03

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **ASM-B1 (UK-only, eu-west-2)** — legal confirmation pending for data residency/subprocessors. |
| **Why it matters** | NFR-S1; AWS subprocessor DPA required. |
| **Recommended action** | Legal review of AWS DPA and log/data locations. |
| **Owner** | Legal / DPO |
| **Status** | open |

### FIND-AS-04

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | **ASM-O2 (limited cloud maturity)** — proposal chooses EC2 for operability; reasonable per [decision matrix](../../decision-matrix.md) simplicity criterion. |
| **Why it matters** | Validates technology choice for team context. |
| **Recommended action** | None — monitor during post-implementation review. |
| **Owner** | Engineering Lead |
| **Status** | open |

---

## 12. Organisational Fit

### FIND-OF-01

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | Proposal **aligns well** with CON-O1 (small team) and CON-O2 (EC2 familiarity) — high [decision matrix](../../decision-matrix.md) scores for simplicity and operational fit **for team with EC2 skills**. |
| **Why it matters** | Positive — if team lacks EC2 skills, finding inverts to P1 risk. |
| **Recommended action** | Confirm ASM-O2 skills assessment; training plan if gaps. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-OF-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | **Organisational capacity vs Tier ambition** — business wants high availability; org may only support business-hours on-call ([organisational constraints](../../organisational-constraints.md)). |
| **Why it matters** | Undocumented principle conflict. |
| **Recommended action** | Align tier, on-call, and alert routing in single signed ops model. |
| **Owner** | Engineering Lead + Commercial Sponsor |
| **Status** | open |

---

## 13. Simplicity vs Completeness

### FIND-SI-01

| Field | Detail |
|-------|--------|
| **Severity** | P2 |
| **Finding** | Proposal **appropriately avoids over-engineering** — no multi-region, ECS, WAF, RDS, CDN in v1. Matches [manifesto](../../manifesto.md) simplicity and CON-F1. |
| **Why it matters** | Positive finding for small team context. |
| **Recommended action** | Resist scope creep during implementation unless P0/P1 forces addition. |
| **Owner** | Engineering Lead |
| **Status** | open |

### FIND-SI-02

| Field | Detail |
|-------|--------|
| **Severity** | P1 |
| **Finding** | Simplicity may **under-deliver** on stated HA if business interprets "high availability" as Tier 1 checkout with 24/7 response — completeness gap is **expectation**, not technology. |
| **Why it matters** | [Principle conflicts](../../principle-conflicts.md): reliability vs cost vs operability triangle unresolved. |
| **Recommended action** | Explicit stakeholder sign-off on what v1 **does not** include. |
| **Owner** | Commercial Sponsor |
| **Status** | open |

---

## 14. Risks Accepted

The following risks are **proposed for acceptance** in ADR if P0 items resolved — **not accepted by this review yet**:

| Risk | Proposal severity | Review disposition |
|------|-------------------|-------------------|
| Single-region (regional AWS failure) | Medium | Acceptable for v1 if FIND-AR-02 documented |
| WAF deferred until pre-sale | Medium | Acceptable if FIND-SE-01 threat model + FIND-SE-02 milestone |
| No RDS / external data plane | Medium | Acceptable if FIND-DD-02 confirmed |
| CloudFront deferred | Low–Med | Acceptable — P2 |
| CloudWatch-only (no APM) | Medium | Acceptable if FIND-RT-03 app metrics committed |
| DNS TTL rollback delay | Medium | Acceptable with cutover rehearsal |
| External API dependency HA | High | **Cannot accept silently** — requires FIND-BA-02 degradation sign-off |

---

## 15. Questions Before Approval

Consolidated from proposal §11 and review findings — **must be answered or explicitly risk-accepted before ADR**:

1. Where does session/basket state live today? (FIND-RT-01, FIND-AS-01)
2. What is finance-approved monthly budget — normal and peak? (FIND-CO-01)
3. What is maximum acceptable complete outage during named sale? (FIND-BA-01)
4. Payment model — confirm no PAN on origin? (FIND-RT-02)
5. Integration inventory — egress, allowlisting, SLAs? (FIND-NW-01)
6. Checkout path tier — Tier 1 or Tier 2? On-call match? (FIND-AR-01, FIND-OF-02)
7. NAT topology decision — single vs dual? (FIND-NW-02)
8. Load test scenario and pass criteria? (FIND-SC-01)
9. Threat model complete — WAF deferral signed? (FIND-SE-01, FIND-SE-02)
10. DNS cutover and rollback rehearsed? (implementation gate)

---

## Review Output (Framework Template)

**System:** ER-001 — Customer-facing e-commerce storefront on AWS  
**Date:** 2026-06-28  
**Reviewers:** Principal Cloud Architect (review document)  
**Outcome:** **Not approved** — conditions below

### P0 — Launch Blockers (open)

| ID | Summary | Owner |
|----|---------|-------|
| FIND-BA-01 | Sale-window outage tolerance undefined | Commercial Sponsor |
| FIND-RT-01 | FR-2 session/basket not designed | Engineering Lead |
| FIND-RT-02 | Payment boundary unvalidated | Engineering Lead |
| FIND-SE-01 | No threat model | Security |
| FIND-NW-01 | Integration reachability unknown | Engineering Lead |
| FIND-CO-01 | No budget envelope | Finance |
| FIND-SC-01 | Peak load / load test undefined | Engineering Lead |
| FIND-AS-01 | ASM-A1 stateless unverified | Engineering Lead |

### P1 — Significant Risks (open)

12 findings — see sections above (FIND-BA-02 through FIND-SI-02). Require mitigation plan or documented acceptance before **production** cutover; several required before **ADR**.

### P2 — Improvement Opportunities (open)

8 findings — backlog with revisit triggers; do not block ADR if P0 cleared.

### ADRs Required

| ADR topic | Trigger |
|-----------|---------|
| Accept v1 architecture pattern (ALB + ASG + EC2 eu-west-2) | After P0 resolution |
| Session store (if needed) | FIND-RT-01 outcome |
| NAT topology | FIND-NW-02 decision |
| WAF deferral accepted risk | FIND-SE-01/02 |
| Tier assignment checkout vs storefront | FIND-AR-01 |

---

## Approval Status

**Not approved yet**

Proposal v1 is a **credible direction** for review iteration. It is **not approved** for ADR acceptance or build until minimum changes below are addressed.

---

## Minimum Changes Required Before ADR

The following must be completed or explicitly recorded as **executive-accepted P0 risk** (with named owner and sign-off) before writing an ADR that accepts architecture v1:

| # | Change | Closes |
|---|--------|--------|
| 1 | **Stateless verification** — document where session/basket lives; confirm externalised or add session-store addendum to proposal | FIND-RT-01, FIND-AS-01, FIND-DD-01 |
| 2 | **Payment flow confirmation** — security sign-off on no PAN at origin | FIND-RT-02 |
| 3 | **Integration inventory** — all external APIs, egress, allowlisting, failure impact | FIND-NW-01, FIND-AS-02, FIND-BA-02 |
| 4 | **Budget range approved** — normal + peak month order-of-magnitude | FIND-CO-01, FIND-CO-02 |
| 5 | **Business HA definition** — minutes of acceptable complete outage during named sale; tier sign-off | FIND-BA-01, FIND-AR-01, FIND-SI-02 |
| 6 | **Lightweight threat model** — assets, actors, controls, WAF deferral residual risk | FIND-SE-01 |
| 7 | **Load test scenario defined** — inputs from analytics; pass/fail criteria (execution may follow ADR) | FIND-SC-01 |

**ADR may proceed** when items 1–7 are **resolved or P0-risk-accepted with sign-off**. ADR must list alternatives considered (fixed EC2, ECS, single-AZ) and consequences per [ADR template](../../architecture-decision-records/adr-template.md).

---

## Items Allowed to Remain as Accepted Risks (in ADR)

These may be documented as **accepted risks with owner and revisit trigger** without blocking ADR — **after P0 minimum changes met**:

| Item | Condition | Revisit trigger |
|------|-----------|-----------------|
| WAF not in v1 | Threat model complete; app rate limiting minimum | 60 days before major sale |
| Single-region / no multi-region DR | Regional RTO/RPO documented (FIND-AR-02) | Business expands geography |
| RDS deferred | ASM-I1 confirmed | Local DB or session requires AWS datastore |
| CloudFront / CDN deferred | Load test latency acceptable | NFR-P1 fail |
| No X-Ray / third-party APM | App metrics + structured logs committed | Checkout MTTR incidents |
| ALB access logs off or short retention | FIND-SE-03 decision recorded | Audit or incident need |
| VPC endpoints deferred | NAT cost accepted | Sustained high NAT cost |
| EC2 vs ECS | EC2 skills confirmed | Operational pain post-90-day review |
| Optional Route 53 health checks | ALB multi-AZ deemed sufficient | Cutover planning |

---

## Items Deferred to Implementation

These are **valid deferrals** — track in implementation backlog, not ADR blockers (unless P0):

| Item | Owner | Gate |
|------|-------|------|
| VPC CIDR and subnet layout | Engineering Lead | Before first resource |
| NAT single vs dual AZ decision | Engineering Lead + Finance | Before prod |
| Exact instance type and ASG min/max | Engineering Lead | After load test |
| CI/CD pipeline and rollback automation | Engineering Lead | Before prod deploy (SC-T4) |
| Runbooks linked to alarms | Engineering Lead | Before cutover |
| CloudWatch alarm thresholds | Engineering Lead | Before cutover |
| Patch/AMI pipeline | Engineering Lead | Before prod |
| Synthetic checkout canary | Engineering Lead | Before first sale |
| Application checkout metrics | Engineering Lead | Before first sale |
| DNS cutover rehearsal | Engineering Lead | Cutover week |
| Game day / recovery test | Engineering Lead | SC-T3 before cutover |
| Tagging enforcement | Engineering Lead | First resource |
| Diagram v2 | Engineering Lead | After ADR |

---

## Positive Observations

Recorded for balance — not approval:

- Proposal v1 is **honest about uncertainty** and aligned with discovery disclaimer.
- Technology choice (ALB + ASG + EC2) scores well on **simplicity and organisational fit** for assumed team profile.
- RDS deferral is **well-reasoned** given external integration assumption.
- Deferral register with revisit triggers follows playbook **engineering judgement** standards.
- Multi-AZ at application tier addresses **instance and AZ failure** within single-region scope appropriately for budget context.

---

## Related Documents

- [08 — Architecture Proposal v1](08-architecture-proposal-v1.md)
- [07 — Architecture Discovery Summary](07-architecture-discovery-summary.md)
- [Architecture Review Framework](../../architecture-review-framework/README.md)
- [Principle Conflicts](../../principle-conflicts.md)
- [Architecture Tiers](../../architecture-tiers.md)

---

*Review v1 complete. Proposal revision (v1.1) or ADR drafting follows P0 disposition.*
