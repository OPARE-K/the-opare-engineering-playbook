# ER-001: Business Problem

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 01 — Business Problem  
**Status:** Discovery  
**Date:** 2026-06-28

---

## Problem Statement

A small UK e-commerce company operates a customer-facing web application that supports product discovery, basket management, checkout, and order confirmation. The application currently runs outside a cloud platform the team is confident operating at scale. The business wants to **migrate that customer-facing web application to a cloud environment** so that availability, operational resilience, and cost predictability improve — without expanding the engineering team.

The immediate business problem is not "move to the cloud." It is: **the current hosting model cannot reliably support revenue-critical traffic during seasonal sales peaks, and the team lacks confidence that the platform will remain available when it matters most.**

---

## Who Is Affected

| Stakeholder | Interest |
|-------------|----------|
| **Customers (UK)** | Browse, purchase, and receive order confirmation without errors or prolonged unavailability — especially during promotions |
| **Commercial / marketing** | Run seasonal campaigns without fear that traffic spikes will take the site offline |
| **Engineering team (small)** | Operate the platform sustainably — deploy changes safely, respond to incidents, control cost |
| **Finance** | Predictable infrastructure spend aligned with a budget-conscious organisation |
| **Customer support** | Fewer incident-driven contacts during peak trading periods |

---

## Current Situation

The following is understood at discovery. Details require validation with the business and engineering team.

- The web application is described as **stateless at the application tier** — session or cart state, if any, is assumed to live outside individual compute instances (to be confirmed in assumptions document).
- Traffic is modest in absolute terms: approximately **5,000 daily users** under normal conditions.
- Traffic is **highly uneven**: seasonal sales create sharp peaks relative to baseline.
- The organisation is **budget-conscious** — infrastructure spend is scrutinised and must be justified against revenue impact.
- The engineering team is **small** — limited capacity for 24/7 specialist operations, complex platform engineering, or large migration programmes.
- **High availability** is a stated requirement — the business treats storefront uptime during trading periods as materially linked to revenue and reputation.

What is **not yet confirmed**: current hosting provider, historical outage frequency, current monthly infrastructure cost, peak conversion rates during sales, and whether checkout is in scope for this migration or handled elsewhere.

---

## Why Migration Is Being Considered Now

Migration is driven by business and operational pressure, not technology preference alone.

1. **Peak-period risk.** Seasonal sales concentrate revenue into short windows. Downtime or severe degradation during those windows has disproportionate commercial impact compared to downtime on a quiet Tuesday.

2. **Operational uncertainty.** The team does not trust the current environment to absorb peak load or recover quickly from failure. That uncertainty constrains marketing's willingness to run aggressive campaigns.

3. **Growth without proportional headcount.** The business wants improved resilience and operability without hiring a dedicated platform team. Any future platform must be operable by the existing team.

4. **Cost visibility.** Finance wants clearer attribution and forecasting of run costs — difficult when the current hosting model lacks granularity or elasticity aligned to demand.

---

## What Happens If We Do Nothing

| Risk | Business impact |
|------|-----------------|
| Outage during seasonal sale | Direct revenue loss, customer churn, reputational damage, support load |
| Slow degradation under peak load | Abandoned baskets, reduced conversion, campaign underperformance |
| Continued operational fragility | Engineering time spent firefighting instead of product improvement |
| Over-provisioning to "be safe" | Wasted spend in quiet periods — unacceptable for a budget-conscious organisation |
| Delayed migration | Technical debt and knowledge concentration in current hosting model intensify |

---

## Scope of This Engineering Review (Discovery Phase)

This document establishes **why the problem exists**. It deliberately excludes:

- Cloud service selection
- Architecture patterns or diagrams
- Implementation plans

Subsequent discovery documents will define functional and non-functional requirements, assumptions, constraints, and success criteria before any architecture proposal is produced.

---

## Open Questions for Stakeholder Validation

1. What specific incidents or near-misses triggered this migration initiative?
2. What percentage of annual revenue occurs during seasonal peak periods?
3. Is the entire checkout and payment path in scope, or only the browsing/catalogue tier?
4. Who is the executive sponsor, and what is the decision deadline?
5. What does "high availability" mean to the business in plain language — minutes of downtime per month, or zero tolerance during advertised sales?

---

## Related Documents

- [02 — Functional Requirements](02-functional-requirements.md)
- [03 — Non-functional Requirements](03-non-functional-requirements.md)
- [04 — Assumptions](04-assumptions.md)
- [05 — Constraints](05-constraints.md)
- [06 — Success Criteria](06-success-criteria.md)
