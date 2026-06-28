# ER-001: Architecture Discovery Summary

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 07 — Architecture Discovery Summary  
**Status:** Discovery complete — ready for stakeholder review  
**Date:** 2026-06-28

---

## Review Purpose

This summary consolidates discovery documents 01–06 for the architecture review meeting **before any architecture proposal**. It states what is known, what is unknown, what must be answered, and which risks exist prior to technology selection.

No cloud services, patterns, or implementation approaches have been chosen.

---

## Problem in One Paragraph

A small UK e-commerce company with approximately 5,000 daily users needs to migrate its stateless customer-facing web application to improve availability and operational confidence during seasonal sales peaks. The organisation is budget-conscious, staffed by a small engineering team, and treats storefront uptime as revenue-critical — without sector-specific regulation beyond standard UK data protection and consumer trading practice. Discovery defines **what** must be achieved; architecture has **not** begun.

---

## What We Know

### Business

- Customer-facing web storefront supports browse → basket → checkout → payment → confirmation.
- Traffic is modest daily but **spikes sharply** during seasonal sales — the primary risk window.
- High availability is a **mandated requirement**; exact downtime tolerance during named sales is **not yet quantified**.
- Organisation is **budget-conscious**; finance expects predictable infrastructure spend.
- **Small engineering team** builds and operates — no dedicated platform function assumed.

### Technical (as stated or strongly inferred)

- Application tier described as **stateless** — externalised session/basket state likely but **unverified**.
- **5,000 daily users** — baseline scale anchor; concurrent and peak load **not measured**.
- Integrations exist for payment, catalogue/inventory, orders, email — **migration scope per integration unknown**.
- UK-primary customers; GBP/VAT context.

### Requirements baseline

- **15 functional requirements** identified (FR-1–FR-9 plus integration boundaries); checkout and payment path is revenue-critical.
- **17 non-functional requirements** proposed with targets — most flagged **validation required**.
- **Proposed tier:** Tier 1 for checkout journey, Tier 2 for browse-only paths — **pending business confirmation**.

### Constraints (accepted)

- Operability within small team capacity limits complexity.
- Seasonal traffic shape demands peak-validated design, not average-day sizing.
- UK GDPR applies; full PCI programme not claimed — payment boundary still constraining.
- Cutover must avoid prolonged maintenance during trading; timeline likely tied to next major sale.

### Success definition (draft)

- 15 success criteria across business, technical, operational, financial, and governance dimensions.
- Explicit failure criteria including prolonged sale outage, order data loss, duplicate payment, and uncontrolled cost overrun.

---

## What We Don't Know

### Critical gaps (block architecture proposal if unresolved)

| Gap | Why it matters |
|-----|----------------|
| **Peak load magnitude** | 5–10× concurrent user estimate is unvalidated — capacity and cost models are speculative |
| **Budget envelope** | No numeric monthly run or migration budget — reliability vs cost trade-off cannot be resolved |
| **"High availability" definition** | No agreed minutes of acceptable outage during named sale vs normal month |
| **Integration migration scope** | Unclear whether payment, OMS, inventory migrate with storefront or remain external |
| **Payment integration model** | Card data boundary unknown — drives security scope |
| **Current baseline metrics** | No p95 latency, error rates, outage history, or support ticket baseline |

### Important gaps (should resolve before design finalisation)

| Gap | Why it matters |
|-----|----------------|
| Exact team size and cloud experience | Operability constraint calibration |
| On-call coverage during sale events | Tier 1 claim vs CON-O3 alignment |
| Migration deadline / next sale date | Scope and phasing |
| Inventory consistency rules during flash sales | Caching and oversell behaviour |
| Current hosting cost baseline | CON-F1 and SC-F1 measurement |
| Admin/back-office in scope or not | Programme boundary |

### Known unknowns (acceptable to carry with documented assumptions)

- Exact promotional calendar 12 months forward — refine before load test scenario sign-off.
- Long-term growth beyond next sale — architecture targets next order of magnitude per playbook principle.

---

## Questions Remaining Unanswered

### For commercial / executive sponsor

1. What duration of **complete storefront outage** during a live advertised sale is unacceptable — 0, 5, 15, or 60 minutes?
2. What percentage of annual revenue occurs during peak sale events?
3. What is the **hard deadline** for migration relative to the next major sale?
4. Is marketing willing to accept **graceful degradation** (browse without checkout) during incident — or is any checkout impairment equivalent to failure?

### For finance

5. What is **approved monthly infrastructure budget** for normal month and peak month?
6. What is current monthly hosting spend (baseline for comparison)?

### For engineering

7. Provide **30-day traffic analytics**: daily users, peak concurrent sessions, last sale peak multiplier.
8. Confirm **stateless** claim — session affinity, local state, file storage, WebSocket usage?
9. Inventory all **integrations** with ownership, protocol, SLA, and IP allowlisting requirements.
10. Describe **payment flow** — redirect, hosted fields, tokenisation; PCI SAQ type if known.
11. Current **deployment frequency**, rollback practice, and last significant outage — root cause and duration?
12. Who is **on-call** for next sale, and what hours are covered?

### For legal / compliance

13. Confirm UK-only processing or international data/subprocessor concerns.
14. Confirm payment compliance obligations beyond standard UK GDPR.

---

## Risks Before Any Architecture Is Proposed

These risks exist **now**, at discovery — architecture cannot eliminate them without stakeholder input; it can only expose them.

| Risk | Severity | Nature |
|------|----------|--------|
| **Peak load underestimated** | High | Capacity failure during first post-migration sale |
| **Budget vs availability conflict unresolved** | High | Design rejected by finance or operated unsafely under cost pressure |
| **HA expectation mismatch** | High | Business expects zero downtime; team capacity and budget support 99.9% |
| **Hidden application state** | High | Stateless migration assumption invalid — scope and timeline explode |
| **Integration fragility** | High | Cloud migration does not fix unreliable payment or inventory APIs |
| **Dual-run cost unbudgeted** | Medium | Parallel environments during cutover exceed CON-F1 |
| **On-call gap during sale** | Medium | Tier 1 NFR with no human response — incidents extend MTTR |
| **Payment boundary unknown** | Medium | Security scope creep or compliance gap at launch |
| **No baseline metrics** | Medium | Success criteria (SC-B3, SC-T2) cannot be measured |
| **Timeline compression** | Medium | Sale deadline forces cut scope or skip DR/load validation |
| **Team cloud maturity** | Medium | Operational failure after successful technical cutover |
| **Oversell during cached inventory** | Low–Med | Commercial incident if ASM-I3 staleness assumption wrong |

---

## Recommended Next Steps (Pre-Architecture)

1. **Stakeholder workshop** — resolve critical gaps: HA definition, budget, sale deadline, integration scope.
2. **Analytics export** — validate ASM-T2, ASM-T3; define load test scenario inputs.
3. **Integration and payment inventory** — document CON-T2 dependencies with failure modes.
4. **Baseline capture** — latency, errors, support tickets, current cost for success criteria measurement.
5. **Tier confirmation** — assign checkout vs browse tiers; reconcile with on-call capacity per [Architecture Tiers](../../architecture-tiers.md).
6. **Architecture proposal phase** — only after workshop outcomes recorded as ADR context or updated assumptions.

---

## Discovery Document Index

| Doc | Title |
|-----|-------|
| [01](01-business-problem.md) | Business Problem |
| [02](02-functional-requirements.md) | Functional Requirements |
| [03](03-non-functional-requirements.md) | Non-Functional Requirements |
| [04](04-assumptions.md) | Assumptions |
| [05](05-constraints.md) | Constraints |
| [06](06-success-criteria.md) | Success Criteria |

---

## Review Meeting Recommendation

**Outcome sought from discovery review:**

- Validated or corrected assumptions marked **Critical** in ASM register
- Numeric targets agreed for NFR-A1, NFR-P2, NFR-R1, NFR-F1 (budget)
- Confirmed integration scope boundary
- Go / no-go to **architecture proposal** phase with updated discovery documents

**Not in scope for this meeting:** technology selection, diagrams, implementation planning.

---

*Discovery demonstrates engineering judgement applied before selecting technology. Architecture begins when the questions above have answers — or documented accepted risks.*
