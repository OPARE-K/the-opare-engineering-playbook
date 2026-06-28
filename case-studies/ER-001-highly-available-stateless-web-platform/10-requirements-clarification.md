# ER-001: Requirements Clarification

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 10 — Requirements Clarification  
**Status:** Workshop preparation — questions open  
**Date:** 2026-06-28  
**Source review:** [09-architecture-review-v1.md](09-architecture-review-v1.md) (P0 findings)

---

## Purpose

Architecture proposal v1 is **not approved**. Eight P0 findings from the architecture review cannot be closed by engineering assumptions alone. This document converts each P0 into **stakeholder questions** for a requirements clarification workshop.

**This document does not answer the questions.** It identifies who must respond, why the response matters, which architectural decisions depend on it, and what risk remains if the answer stays unknown.

**Reference:** [08-architecture-proposal-v1.md](08-architecture-proposal-v1.md) | [07-architecture-discovery-summary.md](07-architecture-discovery-summary.md)

---

## How to Use

1. Assign each question to a named individual before the workshop — not only a role.
2. Request pre-read materials where noted (analytics, payment contract, integration list).
3. Record answers in an workshop output annex — update [04-assumptions.md](04-assumptions.md) and close P0 items in review v1.1.
4. Do not proceed to ADR until every P0 is **resolved** or **executive-accepted** with named sign-off.

---

# Questions by Stakeholder Domain

---

## Business

### P0-01 — Sale-window availability and outage tolerance

**Review reference:** FIND-BA-01  
**Priority:** P0 — blocks ADR and tier assignment

#### Business Question

1. During a **named seasonal sale** (e.g., Black Friday, Boxing Day, summer promotion), what is the maximum acceptable duration of a **complete storefront outage** — where customers cannot browse or checkout at all?
2. Is **partial degradation** acceptable — for example, browse works but checkout is unavailable — and for how long?
3. What percentage of **annual revenue** is concentrated in peak sale windows?
4. Does the business require the **checkout path** to be held to a stricter standard than catalogue browsing?

#### Technical Reason

Infrastructure redundancy, capacity headroom, on-call investment, and operational runbook depth are sized against an availability target. Discovery proposed 99.9% monthly ([NFR-A1](03-non-functional-requirements.md)) and Tier 1 for checkout ([03-non-functional-requirements.md](03-non-functional-requirements.md)) — these numbers are **not equivalent** and neither is signed. Engineering cannot translate "high availability" into design without minutes-of-outage tolerance during revenue-critical events.

#### Decision Impact

- Service tier assignment ([architecture tiers](../../architecture-tiers.md))
- Whether current proposal v1 scope is sufficient or materially undersized
- Auto Scaling maximum capacity and pre-warm strategy before known events
- Investment in synthetic monitoring, on-call coverage, and pre-sale hardening
- Success criteria SC-B1 and SC-B2 — pass/fail definition

#### Possible Outcomes

- Zero tolerance for complete outage during advertised sale hours — implies highest tier expectations and operational investment
- Bounded tolerance (e.g., 5, 15, or 30 minutes complete outage) — enables explicit trade-off documentation
- Stricter checkout than browse — implies split-tier design and differentiated degradation behaviour
- Acceptance of Tier 2-equivalent availability for entire storefront — reduces cost and ops burden; must be explicit

#### Risk if Unknown

Engineering optimises for the wrong failure mode. Business may expect sale-period behaviour that the approved design cannot deliver. Post-migration outage becomes a programme failure and ADR reversal. Reliability vs cost trade-off remains undocumented ([principle conflicts](../../principle-conflicts.md)).

#### Stakeholder

**Primary:** Commercial / Product Sponsor  
**Required input:** Marketing (campaign calendar), Finance (revenue concentration)

---

### P0-07 — Peak traffic and load validation inputs

**Review reference:** FIND-SC-01  
**Priority:** P0 — blocks capacity approval and load test design

#### Business Question

1. What are the **dates and durations** of planned promotional events for the next 12 months?
2. Can the business provide **historical traffic data** from the most recent seasonal sale — sessions, orders, peak hour, concurrent users if available?
3. What **marketing activities** drive peak traffic (email blast, paid ads, influencer) and when do they land relative to sale start?
4. What is the **business impact** if the site is slow but not down during peak — lost revenue estimate or qualitative severity?

#### Technical Reason

Proposal v1 sizes Auto Scaling bounds and instance class using unvalidated assumptions ASM-T2 and ASM-T3 ([04-assumptions.md](04-assumptions.md)). Success criterion SC-T2 requires a load test with defined pass/fail. Engineering cannot define that scenario without business-owned traffic evidence. Daily user count alone does not determine peak concurrent load.

#### Decision Impact

- Auto Scaling minimum, maximum, and pre-warm plan
- Load test profile and go/no-go gate before cutover
- Whether proposal v1 can survive first post-migration sale
- Peak-month cost forecast ([SC-F2](06-success-criteria.md))

#### Possible Outcomes

- Historical analytics available — engineering defines evidence-based load test
- No analytics — business accepts risk of assumed peak multiplier with named executive sign-off
- Peak larger than assumed — proposal v1 requires revision (v1.1) before build
- Peak smaller than assumed — cost optimisation possible; still requires documented evidence

#### Risk if Unknown

First seasonal sale after migration is the load test. Capacity failure during revenue peak is the highest-severity discovery risk ([07-architecture-discovery-summary.md](07-architecture-discovery-summary.md)). Finance cannot approve peak-month budget without a traffic scenario.

#### Stakeholder

**Primary:** Commercial / Marketing  
**Required input:** Engineering (analytics export if exists), Finance (peak revenue context)

---

## Operations

### P0-01-OPS — Operational coverage aligned to availability commitment

**Review reference:** FIND-BA-01 (operational dimension); related FIND-AR-01, FIND-OF-02 (P1, tier/on-call)  
**Priority:** P0 — must align with P0-01 business answers before ADR

#### Business Question

1. Who is **accountable** for production storefront availability — named role, not only team?
2. During the **first major sale** after migration, what **hours** must be covered by someone able to respond within 30 minutes?
3. Is **24/7 paging** required year-round, during sale windows only, or business-hours only?
4. If an alert fires at 2am during a live sale, **who is paged** and is that person empowered to act (rollback, scale, disable feature)?

#### Technical Reason

Proposal v1 configures CloudWatch alarms assuming incident response exists ([NFR-O3](03-non-functional-requirements.md)). Architecture tiers define different monitoring and on-call expectations for Tier 1 vs Tier 2 ([architecture-tiers.md](../../architecture-tiers.md)). Claiming high availability without defined coverage is an organisational principle conflict ([organisational constraints](../../organisational-constraints.md)).

#### Decision Impact

- Alert routing design and severity thresholds
- Whether Tier 1 checkout claims are credible
- Runbook and escalation investment
- Honest ADR consequences — ops model as part of architecture

#### Possible Outcomes

- Sale-window extended on-call with named roster — aligns with bounded Tier 1 during events
- Business-hours only — Tier 1 claims must be downgraded or accepted as risk
- Third-party/on-call service — budget implication for Finance
- No on-call — architecture must not page humans; automation-only recovery bar rises materially

#### Risk if Unknown

Alerts fire with no responder — MTTR equals time until business hours. Sale-period outage extends unnecessarily. Team burnout if on-call assumed but not resourced.

#### Stakeholder

**Primary:** Engineering Lead  
**Required input:** Commercial Sponsor (sale schedule), HR/Leadership (on-call policy if applicable)

---

### P0-07-OPS — Migration deadline and cutover window

**Review reference:** FIND-SC-01 (programme gate); CON-P1, ASM-O4  
**Priority:** P0 — blocks workshop sequencing if hard deadline exists

#### Business Question

1. What is the **hard deadline** for production cutover relative to the next major sale?
2. Is there a **change freeze** period before peak trading when deployment risk is unacceptable?
3. How many weeks of **parallel running** (old and new environment) can the business fund and tolerate operationally?

#### Technical Reason

Load test execution, game day, runbook completion, and dual-run cutover require calendar time. Scope compression forces skipped validation — explicit failure mode in discovery ([07-architecture-discovery-summary.md](07-architecture-discovery-summary.md)).

#### Decision Impact

- Programme phasing — what must be in v1 vs deferred
- Dual-run cost ([FIND-CO-03](09-architecture-review-v1.md) P1)
- Whether P0 items can be resolved before build starts

#### Possible Outcomes

- Deadline before next sale — full validation path required or scope cut with signed risk
- Deadline after next sale — migration risk to current platform during upcoming peak
- Flexible deadline — engineering-led validation schedule

#### Risk if Unknown

Engineering commits to dates that cannot be met; validation skipped under pressure; self-inflicted outage during cutover ([CON-P2](05-constraints.md)).

#### Stakeholder

**Primary:** Commercial / Product Sponsor  
**Required input:** Finance (dual-run budget), Engineering Lead (effort estimate)

---

## Security

### P0-04 — Threat exposure and control expectations

**Review reference:** FIND-SE-01  
**Priority:** P0 — blocks ADR security consequences and WAF deferral sign-off

#### Business Question

1. What assets does the business consider **most valuable to protect** — customer accounts, order data, brand reputation, payment flow integrity, admin access?
2. Who are the **expected adversaries** — opportunistic attackers, credential stuffers, bots scraping pricing, disgruntled insiders?
3. Has the business experienced **security incidents** on the current platform — DDoS, defacement, fraud, credential breach?
4. What is acceptable **residual risk** for a public storefront without edge web application firewall in initial launch?
5. Who **signs off** on accepted security risk — named individual?

#### Technical Reason

Proposal v1 defers WAF with documented residual risk. Deferral requires threat-informed acceptance per [security by design](../../engineering-principles/security-by-design.md) and [engineering workflow](../../engineering-workflow.md) — not absence of analysis. Security controls and logging depth depend on asset value and threat model, which business and security stakeholders own.

#### Decision Impact

- Whether WAF deferral is ADR-acceptable or launch blocker
- Application rate limiting minimum before launch
- Logging and access audit depth
- Security group egress restriction aggressiveness
- Pre-sale security review milestone

#### Possible Outcomes

- Accept documented residual risk until pre-sale review — named signatory and date
- WAF required before any public production traffic — scope and budget change
- Enhanced application controls only — engineering commitment with security sign-off
- Incident history drives prioritised controls — reorder security backlog

#### Risk if Unknown

Security scope undefined. ADR cannot document consequences. Launch proceeds with disputed accountability if incident occurs. Compliance and insurance questions may arise post-incident.

#### Stakeholder

**Primary:** Security lead or accountable senior engineer  
**Required input:** Commercial Sponsor (brand/revenue impact), Engineering Lead (current controls)

---

### P0-03 — Payment flow and cardholder data boundary

**Review reference:** FIND-RT-02  
**Priority:** P0 — blocks ADR security and compliance sections

#### Business Question

1. Which **payment provider** is under contract and who owns the commercial relationship?
2. Does the customer enter card details **only on the payment provider's pages** (redirect/hosted), or does any card data pass through the storefront application or infrastructure?
3. Are **saved payment methods** or wallet features offered — if yes, how are tokens stored?
4. Who last assessed **PCI scope** for the current platform — is there an existing SAQ or provider attestation document?

#### Technical Reason

Proposal v1 assumes redirect or hosted payment so cardholder data does not touch origin ([NFR-S3](03-non-functional-requirements.md)). If PAN transits EC2 or logs, security scope, logging policy, encryption, and assessment obligations change materially. Engineering cannot confirm architecture boundary without payment facts.

#### Decision Impact

- Whether origin infrastructure is in PCI scope
- Logging redaction rules and ALB access log policy
- Security assessment and penetration test scope
- Integration design for payment callbacks and webhooks
- ADR security implications section

#### Possible Outcomes

- Redirect/hosted only — narrower scope; proposal boundary holds
- Hosted fields or API tokenisation on origin — controlled scope with strict controls
- PAN on origin — major architecture and compliance revision required
- Unknown — launch blocker until provider documentation reviewed

#### Risk if Unknown

Accidental PCI scope expansion. Regulatory and contractual exposure. Incorrect logging may capture sensitive authentication data. ADR security section would be inaccurate.

#### Stakeholder

**Primary:** Engineering Lead + Finance (payment contract)  
**Required input:** Payment provider account manager or documentation, Security, Compliance/DPO

---

## Infrastructure

### P0-07-INF — Capacity envelope approval

**Review reference:** FIND-SC-01 (infrastructure sizing gate)  
**Priority:** P0 — linked to P0-07 business traffic questions

#### Business Question

1. Once peak traffic scenario is agreed, does the business approve **temporary scale-out** during sales that increases infrastructure cost for days not months?
2. What is the **maximum acceptable peak-month cost uplift** as a multiplier of a normal month — qualitative or percentage?
3. If load test fails unless capacity is raised, who **approves** additional spend or accepts performance risk?

#### Technical Reason

Auto Scaling maximum and instance sizing translate traffic into infrastructure cost ([NFR-C2](03-non-functional-requirements.md)). Infrastructure design is bounded by finance-approved envelope ([FIND-CO-01](09-architecture-review-v1.md)). Engineering cannot set ASG max without business-cost alignment.

#### Decision Impact

- ASG maximum instance count
- Pre-warming policy before campaigns
- Pass/fail criteria when load test requires higher capacity
- Peak-month forecast submitted to Finance

#### Possible Outcomes

- Approve elastic peak spend within defined multiplier
- Cap max capacity — accept performance degradation or queueing at peak
- Defer sale until capacity investment approved
- Revise proposal v1 upward or downward based on joint traffic/cost decision

#### Risk if Unknown

Auto Scaling capped silently by unspoken budget — site slow or down at peak. Or over-provisioned spend without approval — programme credibility loss with Finance.

#### Stakeholder

**Primary:** Finance  
**Required input:** Commercial Sponsor, Engineering Lead

---

## Finance

### P0-06 — Infrastructure budget envelope

**Review reference:** FIND-CO-01  
**Priority:** P0 — blocks ADR cost implications and build approval

#### Business Question

1. What is the **approved monthly budget** for hosting the customer-facing storefront in a **normal traffic month** after migration?
2. What is the **approved budget** for a **peak sale month** including temporary scale-out?
3. What is the **one-time budget** for migration — dual-run period, tooling, external support if any?
4. What is **current monthly hosting spend** — baseline for comparison ([SC-F1](06-success-criteria.md))?
5. Above what monthly spend threshold is **finance escalation** required?

#### Technical Reason

Proposal v1 lists cost drivers but cannot produce order-of-magnitude approval figures ([ASM-F1](04-assumptions.md)). [Decision matrix](../../decision-matrix.md) cost criterion cannot be evaluated. Reliability vs cost trade-offs ([principle conflicts](../../principle-conflicts.md)) require a budget ceiling to resolve.

#### Decision Impact

- Single vs dual NAT topology (cost vs resilience — P1 FIND-NW-02)
- ASG bounds and instance class
- WAF and enhanced observability timing
- Whether proposal v1 is affordable or requires simplification
- ADR cost implications — build, run, change cost

#### Possible Outcomes

- Defined normal and peak envelopes — engineering sizes within bounds
- Budget below proposal v1 estimate — architecture revision or scope cut required
- Budget aligned with estimate — proceed with tagged forecasting and AWS Budgets
- Migration funded separately — dual-run duration confirmed

#### Risk if Unknown

Design exceeds budget — cut redundancy under operational pressure post-launch. Or under-invest at peak — outage during sale. ADR cost section speculative — finance rejection at gate.

#### Stakeholder

**Primary:** Finance  
**Required input:** Engineering Lead (estimate after traffic scenario), Commercial Sponsor (peak importance)

---

## Application

### P0-02 — Session and basket state ownership

**Review reference:** FIND-RT-01  
**Priority:** P0 — blocks FR-2 traceability and architecture completeness

#### Business Question

1. From a **customer perspective**, how long must a basket persist — same session, 24 hours, 7 days?
2. If basket is lost mid-shop during a sale, is that equivalent to **site down** for success criteria purposes?
3. Are **guest baskets** and **logged-in customer baskets** subject to the same rules?
4. Who owns the **product decision** on basket persistence — product, commercial, or engineering?

#### Technical Reason

Functional requirement FR-2 ([02-functional-requirements.md](02-functional-requirements.md)) mandates basket continuity. Proposal v1 deferred session architecture pending verification. Business rules on persistence duration and severity of loss determine whether external session store is mandatory for migration and how it must behave under failure.

#### Decision Impact

- Whether a session store component is required in architecture v1.1
- Load balancer stickiness — should remain disabled if state externalised
- RPO for session data if store is introduced
- Customer communications during incident ("your basket may be empty")

#### Possible Outcomes

- Short TTL basket acceptable — simpler store or existing backend sufficient
- Long-lived basket mandatory — durable session store required before launch
- Basket loss unacceptable during sale — Tier 1 behaviour for session dependency
- Business accepts basket reset on deploy/failover — engineering documents degradation

#### Risk if Unknown

FR-2 untested against design. Migration breaks conversion silently. Mid-sale deploy clears baskets — commercial incident.

#### Stakeholder

**Primary:** Product / Commercial Sponsor  
**Required input:** Engineering Lead (current behaviour), Customer Support (complaint patterns)

---

### P0-08 — Application statelessness verification

**Review reference:** FIND-AS-01  
**Priority:** P0 — blocks core proposal assumption CON-T1

#### Business Question

1. Is the business aware of any **features** that depend on a specific server being sticky — uploads, admin tools, legacy modules, scheduled jobs on web nodes?
2. Are there **planned features** before next sale that require local server state or file storage on web instances?
3. Who can confirm **authoritatively** that the application tier is stateless — engineering owner name?

#### Technical Reason

Entire proposal v1 assumes stateless compute ([CON-T1](05-constraints.md), ASM-A1). Auto Scaling, rolling deploy, and multi-AZ distribution fail if undeclared local state exists. This is a factual question about the application — but business must confirm scope of "customer-facing web application" included in migration.

#### Decision Impact

- Validity of proposal v1 compute model
- Migration scope boundary — what is migrated vs refactored first
- Whether programme timeline must extend for state externalisation refactor
- ADR assumption register — ASM-A1 true or false

#### Possible Outcomes

- Confirmed stateless — proposal compute model holds
- Local session/files discovered — refactor gate or session store addendum before migration
- Scope excludes legacy module — architectural split (two deployables)
- Unknown until audit complete — build blocked

#### Risk if Unknown

Hidden state causes production defects after scale or deploy. ASM-A1 false invalidates review disposition. Timeline and cost explode if discovered late.

#### Stakeholder

**Primary:** Engineering Lead / Application owner  
**Required input:** Product (scope of migrated application)

---

## Compliance

### P0-03-C — Data protection and subprocessors

**Review reference:** FIND-RT-02 (data boundary); FIND-AS-03 (P1 ASM-B1 legal) — compliance dimension of P0 payment and data questions  
**Priority:** P0 — blocks ADR compliance and NFR-S1 sign-off

#### Business Question

1. Who is the **Data Protection Officer** or accountable person for UK GDPR?
2. Are customers **UK-only**, or do EU/international data subjects use the storefront?
3. Is an **AWS Data Processing Agreement** already in place — who signed it?
4. What **personal data** flows through the storefront — account, address, order history, marketing preferences, IP logs?
5. What **retention periods** apply to customer and order logs?

#### Technical Reason

NFR-S1 requires UK GDPR compliance ([03-non-functional-requirements.md](03-non-functional-requirements.md)). Region selection and logging design depend on lawful processing, retention, and subprocessor documentation. Payment data boundary overlaps PCI and GDPR — cannot be separated in ADR.

#### Decision Impact

- Region and data residency documentation in ADR
- Log retention and redaction policy
- Subprocessor register update for cloud migration
- Whether legal review is migration gate

#### Possible Outcomes

- UK-only, DPA in place — proceed with documented retention and redaction
- International users — data transfer assessment required
- DPA not signed — compliance gate before production
- Retention limits drive CloudWatch and ALB log policy

#### Risk if Unknown

Unlawful processing or retention. ICO exposure. ADR compliance section incomplete. Incident response hampered without data inventory.

#### Stakeholder

**Primary:** DPO / Legal / Compliance  
**Required input:** Engineering Lead (data flows), Finance (vendor contracts)

---

### P0-04-C — Risk acceptance authority

**Review reference:** FIND-SE-01  
**Priority:** P0 — governance gate for deferred controls

#### Business Question

1. Who has **authority to accept residual security risk** for production launch — role and name?
2. Does the organisation require **board or insurer notification** for ecommerce platform changes?
3. Are there **contractual uptime or security obligations** to partners or marketplaces?

#### Technical Reason

Deferred WAF and other controls require documented accepted risk with owner ([architecture review framework](../../architecture-review-framework/README.md)). ADR and review template require sign-off for P0 acceptance — not engineering alone.

#### Decision Impact

- ADR accepted-risk section validity
- Whether launch proceeds with deferrals
- External audit or customer contractual exposure

#### Possible Outcomes

- Named executive accepts documented residual risks
- Compliance mandates controls before launch — scope change
- Contractual SLA requires higher tier — links to P0-01

#### Risk if Unknown

Risks accepted informally — no accountability. Post-incident dispute over who approved launch.

#### Stakeholder

**Primary:** Legal / Compliance / Executive sponsor  
**Required input:** Security, Commercial (partner contracts)

---

## Integration

### P0-05 — External system connectivity and ownership

**Review reference:** FIND-NW-01  
**Priority:** P0 — blocks network design and cutover planning

#### Business Question

1. What is the **complete list** of external systems the storefront calls in production — catalogue, stock, pricing, orders, payment, email, CRM, analytics?
2. For each system: who is the **commercial and technical owner** — internal team or vendor?
3. Do any vendors **restrict source IP addresses** — do they require notification before traffic originates from new cloud egress?
4. What are **contractual SLAs or support hours** for payment and order systems during sales?
5. If an external API is down during a sale, what **customer-facing behaviour** has the business approved — hard error, cached catalogue, queue, maintenance page?

#### Technical Reason

Proposal v1 assumes outbound HTTPS from private subnets to existing APIs ([ASM-I1](04-assumptions.md), [CON-T2](05-constraints.md)). Migration fails if partners block new IP addresses or if connectivity requirements are unknown. End-to-end availability ([FIND-BA-02](09-architecture-review-v1.md) P1) depends on integration SLAs — business owns vendor relationships.

#### Decision Impact

- Egress architecture — fixed IP requirement, allowlisting lead times
- Cutover sequencing — partner notification deadlines
- Graceful degradation design ([NFR-A2](03-non-functional-requirements.md))
- Whether migration improves or merely relocates availability risk
- Programme dependency on third-party change windows

#### Possible Outcomes

- Full inventory with no IP restrictions — simpler egress
- Allowlisting required — lead time and Elastic IP decision before cutover
- Critical vendor without sale support — accepted dependency risk documented
- Missing integration discovered — scope expansion

#### Risk if Unknown

Cutover day connectivity failure. Checkout works in test but fails in prod from blocked egress. Sale lost while AWS infrastructure healthy. Long vendor lead time missed.

#### Stakeholder

**Primary:** Engineering Lead  
**Required input:** Commercial (vendor contracts), each **Integration owner** / vendor account manager, Customer Support (failure UX)

---

### P0-05-INT — Integration migration scope

**Review reference:** FIND-NW-01; ASM-I1, ASM-I2  
**Priority:** P0 — scope boundary for ER-001

#### Business Question

1. Is this programme **storefront hosting only**, or does it include migrating backend systems?
2. Will **payment webhooks** and order callbacks target new endpoints after cutover — who coordinates vendor configuration?
3. Are **sandbox/test environments** available for all integrations from cloud — who approves vendor test access?

#### Technical Reason

ADR scope and proposal v1 assume web tier migration with external backends. If backends must move simultaneously, programme architecture and timeline change fundamentally. Callback URL changes require vendor coordination — not engineering-only.

#### Decision Impact

- ER-001 programme boundary
- RDS and data layer deferral validity
- Cutover runbook complexity
- Number of ADRs and workstreams

#### Possible Outcomes

- Storefront only — proposal scope confirmed
- Backend migration in scope — ER-001 requires rechartering
- Phased — storefront first with explicit backend dependency map
- Vendor test access delayed — migration timeline slips

#### Risk if Unknown

Scope creep or missed dependency. ADR describes wrong system boundary. Cutover breaks order fulfilment or payment confirmation.

#### Stakeholder

**Primary:** Product / Programme sponsor  
**Required input:** Engineering Lead, Finance (scope cost), Integration owners

---

## P0 Summary Matrix

| ID | Review finding | Primary stakeholder domain | Blocks ADR |
|----|----------------|---------------------------|------------|
| P0-01 | FIND-BA-01 | Business | Yes |
| P0-01-OPS | FIND-BA-01 (ops) | Operations | Yes |
| P0-02 | FIND-RT-01 | Application / Business | Yes |
| P0-03 | FIND-RT-02 | Security | Yes |
| P0-03-C | FIND-RT-02 (compliance) | Compliance | Yes |
| P0-04 | FIND-SE-01 | Security | Yes |
| P0-04-C | FIND-SE-01 (governance) | Compliance | Yes |
| P0-05 | FIND-NW-01 | Integration | Yes |
| P0-05-INT | FIND-NW-01 (scope) | Integration / Business | Yes |
| P0-06 | FIND-CO-01 | Finance | Yes |
| P0-07 | FIND-SC-01 | Business | Yes |
| P0-07-INF | FIND-SC-01 (capacity) | Infrastructure / Finance | Yes |
| P0-07-OPS | FIND-SC-01 (timeline) | Operations / Business | Yes |
| P0-08 | FIND-AS-01 | Application | Yes |

*Eight review P0 IDs expand to fourteen question sets where business, compliance, and operations dimensions were implicit in the original finding.*

---

# Requirements Clarification Workshop Agenda

## Workshop title

**ER-001 Requirements Clarification — P0 Closure before Architecture Review v2**

## Duration

**90 minutes** recommended — 120 minutes if integration inventory is reviewed live.

## Objectives

1. **Close or executive-accept** all eight P0 findings from [09-architecture-review-v1.md](09-architecture-review-v1.md).
2. Produce **named, dated answers** — not directional agreement — for availability, budget, payment boundary, integration inventory, traffic peak scenario, and stateless/session model.
3. Update **assumption register** ([04-assumptions.md](04-assumptions.md)) — mark validated or invalidated with evidence reference.
4. Confirm **programme scope** — storefront only vs broader migration.
5. Assign **owners and due dates** for any answer requiring follow-up after workshop.
6. Decide **go / no-go** to ADR drafting and proposal v1.1 revision.

## Required attendees

| Attendee | Role | Mandatory topics |
|----------|------|------------------|
| Commercial / Product Sponsor | Business authority | P0-01, P0-07, degradation UX, tier expectations |
| Marketing representative | Traffic and campaigns | P0-07 peak dates and historical analytics |
| Finance | Budget approval | P0-06, P0-07-INF |
| Engineering Lead | Technical facts and estimates | P0-02, P0-05, P0-08, integration inventory |
| Application owner / senior developer | Statelessness and session | P0-02, P0-08 |
| Security accountable | Threat and payment boundary | P0-03, P0-04 |
| DPO / Legal / Compliance (or delegate) | GDPR, risk acceptance | P0-03-C, P0-04-C |
| Customer Support lead (optional but valuable) | Incident and basket-loss impact | P0-02 |
| Principal architect / review chair | Facilitation, no decision ownership | All — records outcomes |
| Executive sponsor (optional) | P0 executive acceptance only if needed | Risk sign-off |

**Minimum quorum:** Commercial Sponsor + Finance + Engineering Lead + Security + DPO/Legal delegate. Workshop outputs without quorum are **draft only**.

## Pre-work (due 48 hours before workshop)

| Owner | Deliverable |
|-------|-------------|
| Marketing / Commercial | Last sale traffic summary or explicit "data unavailable" |
| Finance | Current hosting cost; draft budget range if exists |
| Engineering | Draft integration list; payment flow diagram from current system |
| Security | Draft asset list for threat discussion |
| Payment provider docs | SAQ or integration guide if held by Finance/Legal |

## Agenda

| Time | Item | Output |
|------|------|--------|
| 0:00 | Context — proposal not approved; P0 purpose | Shared understanding |
| 0:10 | **P0-01** Sale availability and outage tolerance | Minutes tolerance; browse vs checkout; tier intent |
| 0:20 | **P0-01-OPS** On-call and sale coverage | Named roster or explicit gap |
| 0:25 | **P0-06 / P0-07-INF** Budget normal and peak | Approved range or escalation path |
| 0:35 | **P0-07** Peak traffic and sale calendar | Traffic scenario ID for load test |
| 0:45 | **P0-02 / P0-08** Session, basket, statelessness | FR-2 rules; ASM-A1 status |
| 0:55 | **P0-03 / P0-03-C** Payment and data protection | PAN boundary; DPA status |
| 1:05 | **P0-05 / P0-05-INT** Integration inventory and scope | Signed integration list; IP allowlist actions |
| 1:15 | **P0-04 / P0-04-C** Threat model inputs and risk authority | Residual risk signatory |
| 1:20 | **P0-07-OPS** Migration deadline and cutover | Programme dates |
| 1:25 | Open actions, go/no-go ADR | Action log with owners |
| 1:30 | Close | Workshop minutes issued |

## Expected outputs

1. **Workshop minutes** — question, answer, owner, evidence link.
2. **Updated assumption register** — ASM-A1, ASM-B3, ASM-F1, ASM-I1, ASM-T2, ASM-T3 status.
3. **P0 disposition table** — resolved | executive-accepted risk (with signatory) | open with due date.
4. **Load test scenario ID** — named profile for SC-T2 even if test not yet run.
5. **Integration action list** — vendor notifications, allowlisting, callback URL changes with owners.
6. **Go / no-go** to ADR and [08-architecture-proposal-v1.md](08-architecture-proposal-v1.md) revision (v1.1).

## Architecture decisions unlocked by the meeting

When P0 items are resolved, the following decisions may proceed to ADR and proposal v1.1 — **not before**:

| Decision | Unlocked by |
|----------|-------------|
| Accept or revise v1 compute pattern (stateless multi-AZ autoscaling web tier) | P0-08, P0-02 |
| Service tier and availability target for checkout vs browse | P0-01, P0-01-OPS |
| Auto Scaling bounds and peak capacity model | P0-07, P0-06, P0-07-INF |
| Session/basket architecture addendum or confirmation of external store | P0-02, P0-08 |
| Payment and compliance boundary in ADR | P0-03, P0-03-C |
| WAF deferral or mandate in ADR accepted risks | P0-04, P0-04-C |
| Egress and partner connectivity design | P0-05 |
| ER-001 programme scope boundary | P0-05-INT |
| Cost model in ADR (normal vs peak) | P0-06 |
| Load test pass/fail criteria | P0-07 |
| Migration cutover window and dual-run duration | P0-07-OPS |
| Graceful degradation behaviour when dependencies fail | P0-01, P0-05 |
| Observability and on-call alignment | P0-01-OPS |

## Questions explicitly out of workshop scope

- Technology selection debate beyond confirming v1 direction — deferred to post-P0 architecture review v2.
- Terraform, implementation, and diagram authoring.
- P1 and P2 findings — backlog unless time permits; not workshop success criteria.

---

## Related Documents

- [09 — Architecture Review v1](09-architecture-review-v1.md)
- [08 — Architecture Proposal v1](08-architecture-proposal-v1.md)
- [07 — Architecture Discovery Summary](07-architecture-discovery-summary.md)
- [04 — Assumptions](04-assumptions.md)
- [06 — Success Criteria](06-success-criteria.md)

---

*Workshop preparation complete. Answers belong in workshop minutes — not in this document.*
