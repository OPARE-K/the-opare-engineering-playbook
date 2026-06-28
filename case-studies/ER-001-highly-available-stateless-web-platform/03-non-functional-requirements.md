# ER-001: Non-Functional Requirements

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 03 — Non-Functional Requirements  
**Status:** Discovery  
**Date:** 2026-06-28

---

## Purpose

Non-functional requirements (NFRs) define **how the system must behave** — availability, performance, security posture, recoverability, and operability. They constrain future architecture more tightly than functional requirements because they quantify failure tolerance and operational burden.

Each NFR is **justified** against the business problem. Where precise numbers are not yet validated, a **proposed target** is stated with a **validation required** flag.

---

## Service Tier Classification (Proposed)

Based on discovery inputs — customer-facing, revenue-linked, high availability required — the storefront path is proposed as **Tier 1 for the checkout journey** and **Tier 2 for browse-only degradation paths** until the business confirms otherwise.

See [Architecture Tiers](../../architecture-tiers.md) for tier definitions. Final tier assignment is a stakeholder decision, not an architecture decision.

---

## Availability

### NFR-A1: Storefront availability during trading hours

**Requirement:** The customer-facing storefront must maintain availability sufficient to support continuous trading during UK business hours (07:00–22:00 GMT/BST, seven days per week) with minimal unplanned outages.

**Proposed target:** 99.9% monthly availability (~43 minutes unplanned downtime per month) as a starting negotiation point.

**Stricter target for advertised sales windows:** Availability during named promotional events (e.g., Black Friday, Boxing Day, summer sale) must meet business-defined zero-tolerance policy for **complete** outage — to be quantified.

**Justification:** The business stated high availability as a requirement. Revenue and reputation damage during seasonal peaks exceeds the cost of modest redundancy — subject to budget constraint validation.

**Validation required:** Business must define acceptable downtime in minutes during a named sale vs normal trading.

**Influence on future architecture:** Availability target drives redundancy model, deployment strategy, dependency failure behaviour, and on-call expectations.

---

### NFR-A2: Graceful degradation

**Requirement:** Partial failure must not present as a complete site outage where alternative paths exist. Example: catalogue browsing may remain available if a non-critical auxiliary service fails, but checkout must not silently fail or duplicate orders.

**Justification:** True 100% availability is not achievable; controlled degradation preserves partial revenue and customer trust.

**Validation required:** Which features are essential vs deferrable during incident — rank FR-1 through FR-7.

**Influence on future architecture:** Defines fault isolation boundaries and dependency criticality classification.

---

## Performance and Responsiveness

### NFR-P1: Page responsiveness under normal load

**Requirement:** Primary customer interactions (catalogue page, product detail, basket update) must complete within a perceived responsive threshold under normal daily traffic.

**Proposed target:** 95th percentile end-to-end response time under **2 seconds** for server-rendered or API-backed page interactions, excluding third-party payment redirect latency.

**Justification:** E-commerce conversion correlates with latency. 5,000 daily users implies moderate baseline load — responsiveness should be achievable without extreme optimisation, but targets must exist for validation.

**Validation required:** Current baseline p95 latency from existing environment; definition of "page" (SSR vs SPA).

**Influence on future architecture:** Drives caching strategy, geographic placement relative to UK users, and capacity headroom — not specific technologies at this stage.

---

### NFR-P2: Performance under peak load

**Requirement:** During seasonal sales peaks, response time degradation must remain bounded — the site must not become effectively unusable due to latency alone without triggering capacity or incident response.

**Proposed target:** p95 latency no worse than **2× normal baseline** during defined peak (e.g., peak hour of peak sale day).

**Proposed peak multiplier:** **5× to 10×** normal concurrent user load during seasonal events — assumption pending historical analytics.

**Justification:** Peak traffic is the stated risk driver. Architecture must be validated against peak, not average daily users.

**Validation required:** Historical peak concurrent users or sessions; largest observed traffic multiplier during prior sales.

**Influence on future architecture:** Defines load test scenarios, minimum headroom, auto-scaling triggers, and queue/async boundaries.

---

### NFR-P3: Throughput baseline

**Requirement:** The platform must sustain baseline daily traffic of approximately **5,000 unique daily users** without manual intervention.

**Derived estimate (assumption):** Average **50–150 concurrent users** at normal peak hour; **250–1,500 concurrent users** during seasonal peak if 5–10× multiplier holds.

**Justification:** Order-of-magnitude capacity planning requires converting daily users to concurrent load. Daily user count alone is insufficient for architecture sizing.

**Validation required:** Analytics on sessions per day, peak hour concentration, and concurrent session counts.

**Influence on future architecture:** Initial capacity sizing and cost modelling depend on this conversion.

---

## Reliability and Recoverability

### NFR-R1: Recovery Time Objective (RTO)

**Requirement:** Unplanned loss of storefront serving capacity must be recoverable within a business-defined maximum time.

**Proposed target:** **4 hours RTO** for full storefront restoration under Tier 2 framing; **1 hour RTO** if Tier 1 is confirmed for checkout path.

**Justification:** Small team cannot support complex manual recovery during a live sale. Recovery must be procedural and rehearsed.

**Validation required:** Business tolerance for outage duration during a live promotional event.

**Influence on future architecture:** Determines warm standby vs rebuild-from-backup vs active-active posture.

---

### NFR-R2: Recovery Point Objective (RPO)

**Requirement:** Data loss tolerances must be defined for commerce-critical data — orders, payment records, basket state.

**Proposed target:** **15 minutes to 1 hour RPO** for order and transaction data depending on tier confirmation; basket/session loss acceptable within session TTL for non-completed checkouts.

**Justification:** Order and payment data loss has legal and commercial consequences under UK consumer practice. Basket loss mid-checkout is a conversion failure.

**Validation required:** Which data stores are in migration scope; regulatory interpretation for order record retention.

**Influence on future architecture:** Drives backup frequency, replication, and state externalisation design.

---

### NFR-R3: Deployment safety

**Requirement:** Production deployments during trading hours must not cause preventable full-site outage. Rollback must be possible within minutes.

**Proposed target:** Rollback capability within **15 minutes** of a detected bad deployment.

**Justification:** Small teams deploy infrequently but under business pressure during campaigns. Unsafe deploy mechanics are a leading cause of self-inflicted outages.

**Influence on future architecture:** Shapes release strategy, environment parity, and change windows — independent of specific tooling.

---

## Security and Data Protection

### NFR-S1: UK data protection compliance

**Requirement:** Personal data (names, addresses, email, order history, authentication credentials) must be handled in accordance with UK GDPR and the Data Protection Act 2018 — lawful basis documented, retention defined, subject access and deletion supported operationally.

**Justification:** Standard UK business practice applies to all UK e-commerce handling personal data. Not optional regardless of industry-specific regulation.

**Validation required:** Data Processing Agreements with subprocessors; current lawful basis documentation; data retention policy.

**Influence on future architecture:** Data residency preferences, logging redaction, encryption expectations, and vendor/subprocessor assessment.

---

### NFR-S2: Transport and access security

**Requirement:** Customer-facing endpoints must enforce encrypted transport. Administrative access must be authenticated, authorised, and auditable.

**Justification:** Baseline security hygiene expected by customers, payment partners, and insurers. Reduces breach blast radius.

**Influence on future architecture:** Certificate management, identity model for operators, secrets handling — without prescribing implementation.

---

### NFR-S3: Payment data handling boundary

**Requirement:** Cardholder data must not be stored or processed outside the scope agreed with the payment provider's compliance model (e.g., redirect, hosted fields, tokenisation).

**Justification:** Payment scope drives compliance burden and architecture boundary even when full PCI-DSS audit is not claimed.

**Validation required:** Current payment integration model; SAQ type or provider attestation.

**Influence on future architecture:** Determines whether storefront touches raw card data or only payment tokens/sessions.

---

## Operability

### NFR-O1: Operability by small team

**Requirement:** The platform must be operable by the existing small engineering team without requiring dedicated 24/7 platform specialists.

**Proposed interpretation:** Managed operational burden — patching, scaling, certificate renewal, backup verification — must fit within team capacity alongside feature work.

**Justification:** Stated constraint. Architecture that demands continuous specialist attention will fail operationally regardless of theoretical availability.

**Influence on future architecture:** Strong bias toward operational simplicity vs maximum flexibility — to be resolved in trade-off analysis later.

---

### NFR-O2: Incident detectability

**Requirement:** Customer-impacting failures must be detected before widespread user reports where possible.

**Proposed target:** Detection within **5 minutes** of checkout failure rate exceeding defined threshold during trading hours.

**Justification:** MTTR is bounded by MTTD. Small team cannot rely on social media as monitoring.

**Influence on future architecture:** Defines minimum observability requirements and alert routing to available on-call coverage.

---

### NFR-O3: On-call coverage model

**Requirement:** On-call coverage during peak sales events must be explicitly defined and resourced.

**Proposed baseline:** Business-hours primary coverage with extended hours or on-call during named sales — not assumed 24/7 unless business confirms.

**Justification:** High availability NFRs are meaningless without human response capacity. See [Organisational Constraints](../../organisational-constraints.md).

**Validation required:** Who is paged; coverage during overnight sale launches.

**Influence on future architecture:** Tier 1 NFRs with business-hours on-call only is a **principle conflict** requiring documented acceptance.

---

## Cost and Efficiency

### NFR-C1: Cost predictability

**Requirement:** Monthly infrastructure run cost must be forecastable within an agreed tolerance and visible to finance.

**Proposed target:** Monthly variance **≤ 20%** from approved forecast under normal traffic; peak month identified in advance with separate forecast.

**Justification:** Budget-conscious organisation cannot absorb unbounded utility billing surprises.

**Influence on future architecture:** Drives tagging, budget alerts, capacity ceilings, and trade-offs between always-on redundancy vs elasticity.

---

### NFR-C2: Cost alignment to demand

**Requirement:** Infrastructure spend should scale with traffic where economically rational — avoid paying for peak capacity year-round unless justified by RTO/RPO.

**Justification:** Seasonal traffic pattern makes flat over-provisioning financially unacceptable.

**Influence on future architecture:** Elastic vs fixed capacity trade-off; influences peak headroom strategy.

---

## Observability (Minimum)

### NFR-OB1: Business-visible metrics

**Requirement:** The team must be able to answer during an incident: Are customers completing checkout? What is error rate by journey step? What changed recently?

**Justification:** Symptom-based response per playbook observability principles. Infrastructure metrics alone are insufficient for e-commerce incidents.

**Influence on future architecture:** Defines required SLIs — checkout success rate, order submission errors, latency by journey — without selecting tooling.

---

## NFR Summary Table

| ID | Category | Proposed target | Validated? |
|----|----------|-----------------|------------|
| NFR-A1 | Availability | 99.9% monthly; stricter for named sales | No |
| NFR-A2 | Degradation | Defined essential journeys | No |
| NFR-P1 | Latency (normal) | p95 < 2s | No |
| NFR-P2 | Latency (peak) | p95 ≤ 2× baseline | No |
| NFR-P3 | Throughput | 5k daily users; peak concurrent TBC | No |
| NFR-R1 | RTO | 1–4 hours (tier dependent) | No |
| NFR-R2 | RPO | 15 min – 1 hour orders | No |
| NFR-R3 | Deploy rollback | ≤ 15 minutes | No |
| NFR-S1 | UK GDPR | Compliant handling | Partial |
| NFR-S2 | Transport/access | Encrypted, auditable admin | Yes (baseline) |
| NFR-S3 | Payment boundary | Provider-scoped card handling | No |
| NFR-O1 | Team operability | Small team sustainable | Stated |
| NFR-O2 | Detection | ≤ 5 minutes checkout failures | No |
| NFR-O3 | On-call | Named sales coverage TBC | No |
| NFR-C1 | Cost predictability | ≤ 20% forecast variance | No |
| NFR-C2 | Elastic cost | Scale with seasonal demand | Stated |
| NFR-OB1 | SLIs | Checkout success, journey errors | No |

---

## Related Documents

- [02 — Functional Requirements](02-functional-requirements.md)
- [04 — Assumptions](04-assumptions.md)
- [05 — Constraints](05-constraints.md)
- [06 — Success Criteria](06-success-criteria.md)
