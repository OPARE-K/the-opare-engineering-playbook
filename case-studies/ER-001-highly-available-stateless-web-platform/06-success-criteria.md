# ER-001: Success Criteria

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 06 — Success Criteria  
**Status:** Discovery  
**Date:** 2026-06-28

---

## Purpose

Success criteria define **how we know the migration succeeded** — measurable outcomes tied to business and engineering goals. They are agreed before architecture proposal so that design reviews cannot redefine success retrospectively.

Criteria are **justified** against the business problem. Some depend on validation of assumptions and NFR targets.

---

## Success Horizon

| Horizon | Timeframe | Focus |
|---------|-----------|-------|
| **Migration complete** | At production cutover | Safe transition, functional parity, no critical regressions |
| **First seasonal sale** | First peak event post-migration | Availability, performance, and operability under real peak |
| **Steady state** | 90 days post-cutover | Cost predictability, operational sustainability, team confidence |

---

## Business Success Criteria

### SC-B1: Revenue path protected during peak trading

**Criterion:** During the first major seasonal sale post-migration, the storefront completes checkout journeys without a **prolonged complete outage** — defined as continuous unavailability of the full customer journey exceeding the business-validated threshold (proposed: **15 minutes** during advertised sale hours).

**Justification:** Primary business driver for migration. Partial degradation may be acceptable if checkout remains functional — complete outage during paid traffic is failure.

**Measurement:** Checkout success rate, synthetic journey checks, incident timeline, revenue vs forecast during sale window.

**Depends on:** ASM-B3 validation, NFR-A1, marketing event schedule.

---

### SC-B2: Marketing confidence to run campaigns at scale

**Criterion:** Commercial team confirms willingness to execute planned seasonal campaigns on the new platform without requesting infrastructure freeze or offloading traffic to "safe" periods solely due to platform fear.

**Justification:** Migration succeeds only if it removes the organisational constraint that limited campaign ambition.

**Measurement:** Stakeholder sign-off post-first sale; campaign plan for next 12 months unchanged due to platform concerns.

**Depends on:** SC-B1, SC-O2.

---

### SC-B3: No material increase in customer support incident volume attributable to platform

**Criterion:** Customer support ticket volume related to "site down," "checkout error," or "payment failed" during the 30 days following cutover does not exceed **125% of baseline** from equivalent period pre-migration, excluding known campaign effects.

**Justification:** Customer-visible stability is the ultimate availability metric. Internal uptime percentages that do not reflect support load are insufficient.

**Measurement:** Support ticket categorisation; comparison normalised for traffic volume.

**Depends on:** Baseline support metrics export.

---

## Technical Success Criteria

### SC-T1: Functional parity with current storefront

**Criterion:** All **Must have** functional requirements (FR-1 through FR-7, FR-9) verified by agreed test plan before production cutover.

**Justification:** Migration is not an opportunity to drop revenue-critical capability without explicit scope reduction signed by business.

**Measurement:** Traceable test results mapped to FR IDs; signed acceptance by product/commercial owner.

**Depends on:** Integration scope confirmation (ASM-I1).

---

### SC-T2: Performance under validated peak scenario

**Criterion:** Load test representing validated peak concurrent load (ASM-T3) demonstrates p95 journey latency within **NFR-P2** bounds and no unbounded error rate increase.

**Justification:** 5,000 daily users is misleading without peak validation. Success requires evidence at expected worst case, not average day.

**Measurement:** Load test report with scenario definition, metrics, and pass/fail against NFR-P1 and NFR-P2.

**Depends on:** ASM-T2, ASM-T3 validation.

---

### SC-T3: Recovery validated by rehearsal

**Criterion:** Documented recovery procedure for loss of primary serving capacity executed successfully in non-production or controlled test within **NFR-R1 RTO** window.

**Justification:** High availability on paper without tested recovery is an assumption, not success.

**Measurement:** Game day or DR exercise record with timestamps and outcome.

**Depends on:** Final RTO tier assignment.

---

### SC-T4: Safe deployment demonstrated

**Criterion:** At least one production deployment performed with rollback tested — rollback completes within **NFR-R3** (15 minutes) without manual heroics.

**Justification:** Self-inflicted outage during deployment is a leading migration failure mode for small teams.

**Measurement:** Deployment record; rollback drill evidence.

---

## Operational Success Criteria

### SC-O1: Operable by existing team without escalation

**Criterion:** For 90 days post-cutover, **no incident** requires external consultant or vendor professional services to restore service — routine incidents resolved using runbooks and in-house skills.

**Justification:** CON-O1 and NFR-O1. Platform that requires permanent external dependency has failed operability constraint.

**Measurement:** Incident log review at 90 days; runbook usage and gaps documented.

---

### SC-O2: Incident detection meets target

**Criterion:** Customer-impacting incidents during trading hours detected within **NFR-O2** (5 minutes) for checkout failure threshold breaches — verified in at least one game day or real incident.

**Justification:** Undetected outages during sales violate high availability intent regardless of eventual recovery.

**Measurement:** MTTD from incident timeline; alert effectiveness review.

---

### SC-O3: On-call sustainability

**Criterion:** On-call engineers report paging volume and alert quality as **sustainable** through first seasonal sale — no unbounded alert fatigue; actionable alerts linked to runbooks.

**Justification:** Unsustainable on-call leads to attrition and ignored pages — latent availability failure.

**Measurement:** Post-sale retrospective; alert count and signal-to-noise review.

**Depends on:** CON-O3, on-call roster for sale.

---

## Financial Success Criteria

### SC-F1: Run cost within approved envelope

**Criterion:** Monthly infrastructure run cost for **normal traffic month** falls within finance-approved budget range agreed before cutover.

**Justification:** CON-F1. Technical success that bankrupts budget is organisational failure.

**Measurement:** Cost report with tagging; comparison to approved forecast.

**Depends on:** ASM-F1 validation.

---

### SC-F2: Peak month forecast accuracy

**Criterion:** Infrastructure spend during first peak sale month is within **±20% of pre-event forecast** (NFR-C1).

**Justification:** Budget-conscious organisation must predict peak cost, not only baseline.

**Measurement:** Pre-event forecast document vs actual bill attributed to storefront workload.

---

### SC-F3: No emergency unbudgeted spend to restore service

**Criterion:** No finance exception required for unplanned infrastructure scaling or licence purchase to recover from migration-related incident during first 90 days.

**Justification:** Emergency spend indicates architecture or planning failure, not normal variance.

**Measurement:** Finance confirmation; incident cost attribution.

---

## Documentation and Governance Success Criteria

### SC-D1: Decisions recorded

**Criterion:** Significant migration decisions captured in ADRs with alternatives, trade-offs, and accepted risks — minimum set defined at architecture proposal stage.

**Justification:** Playbook standard. Migration knowledge must survive the project.

**Measurement:** ADR index complete and linked from ER-001.

---

### SC-D2: Post-implementation review completed

**Criterion:** Post-implementation review conducted within **90 days** of cutover, updating ADRs with production findings per [ADR template](../../architecture-decision-records/adr-template.md).

**Justification:** [Engineering Workflow](../../engineering-workflow.md) closure. Success includes learning, not only launch.

**Measurement:** Review document published; ADR review date fields populated.

---

## Failure Criteria (Explicit)

The migration is **not successful** if any of the following occur without documented acceptance prior to cutover:

- Prolonged complete storefront outage during first named sale exceeding SC-B1 threshold
- Data loss of confirmed orders exceeding NFR-R2
- Payment duplicate charge incident attributable to migration
- Inability to rollback deployment within NFR-R3 during cutover window
- Run cost exceeding approved envelope by **>30%** in normal month without architectural justification

---

## Success Criteria Summary

| ID | Category | Measurable? | Blocked by validation? |
|----|----------|-------------|------------------------|
| SC-B1 | Business | Yes | Yes — outage threshold |
| SC-B2 | Business | Qualitative | Yes |
| SC-B3 | Business | Yes | Yes — baseline tickets |
| SC-T1 | Technical | Yes | Partial |
| SC-T2 | Technical | Yes | Yes — peak load |
| SC-T3 | Technical | Yes | Yes — RTO tier |
| SC-T4 | Technical | Yes | No |
| SC-O1 | Operational | Yes | No |
| SC-O2 | Operational | Yes | No |
| SC-O3 | Operational | Qualitative | Partial |
| SC-F1 | Financial | Yes | Yes — budget number |
| SC-F2 | Financial | Yes | Yes — forecast |
| SC-F3 | Financial | Yes | No |
| SC-D1 | Governance | Yes | No |
| SC-D2 | Governance | Yes | No |

---

## Related Documents

- [01 — Business Problem](01-business-problem.md)
- [03 — Non-functional Requirements](03-non-functional-requirements.md)
- [05 — Constraints](05-constraints.md)
- [07 — Architecture Discovery Summary](07-architecture-discovery-summary.md)
