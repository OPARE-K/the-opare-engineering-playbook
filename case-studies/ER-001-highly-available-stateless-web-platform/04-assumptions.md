# ER-001: Assumptions

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 04 — Assumptions  
**Status:** Discovery  
**Date:** 2026-06-28

---

## Purpose

Assumptions are beliefs treated as true until validated. Unvalidated assumptions are **pre-architecture risks** — future design may be wrong if they are false.

Each assumption states **impact if wrong** so reviewers can prioritise validation work.

---

## Traffic and Usage Assumptions

### ASM-T1: Daily user volume

**Assumption:** Approximately **5,000 unique users per day** under normal conditions, UK market primarily.

**Source:** Stakeholder brief.

**Impact if wrong:** Capacity and cost models skew. If actual traffic is 5× higher, peak concurrency and database load assumptions collapse. If lower, over-engineering risk increases against budget constraint.

**Validation:** 30-day analytics export — unique visitors, sessions, page views.

---

### ASM-T2: Peak traffic multiplier

**Assumption:** Seasonal sales produce **5× to 10×** normal peak-hour concurrent load, concentrated in **2–4 hour windows**, occurring **2–4 times per year** plus ad-hoc campaigns.

**Source:** Inferred from "peak traffic during seasonal sales" — not validated against historical data.

**Impact if wrong:** If peaks are 20× or sustained over 24–48 hours, NFR-P2 and NFR-C2 cannot be met without architectural scale not yet justified. If peaks are mild, budget spent on peak headroom is wasted.

**Validation:** Prior year sale traffic graphs; marketing calendar for next 12 months.

---

### ASM-T3: Concurrent user estimate

**Assumption:** Normal peak hour concurrent sessions: **50–150**. Seasonal peak: **250–1,500**. Derived from ASM-T1 and ASM-T2 using typical e-commerce session concentration ratios.

**Source:** Engineering estimate — not measured.

**Impact if wrong:** All performance and capacity NFRs rest on this conversion. Wrong order of magnitude invalidates load test design.

**Validation:** Existing analytics: peak concurrent sessions or reconstruct from session starts and average duration.

---

### ASM-T4: Checkout concentration

**Assumption:** A higher proportion of peak traffic converts to checkout during sales than during normal periods — checkout path receives disproportionate load and failure impact.

**Source:** E-commerce domain norm.

**Impact if wrong:** If browsing dominates peak without checkout spike, degradation priorities may be mis-ordered.

**Validation:** Funnel analytics for last seasonal event — browse to basket to checkout conversion by hour.

---

## Application Architecture Assumptions

### ASM-A1: Stateless application tier

**Assumption:** Application compute instances hold **no durable customer state locally**. Session, basket, or cache state — if present — resides in external stores or client tokens.

**Source:** Stakeholder brief.

**Impact if wrong:** If state is pinned to instances, horizontal scaling and rolling deploy assumptions fail. Migration complexity increases materially.

**Validation:** Application architecture review — session affinity, local cache, file uploads, WebSocket state.

---

### ASM-A2: Monolithic or modular monolith storefront

**Assumption:** The customer-facing application is a **single deployable unit** (monolith or well-bounded modular monolith), not a large microservice estate.

**Source:** Inferred from small team and moderate scale.

**Impact if wrong:** If many independent services exist, migration scope, integration testing, and operational burden expand significantly.

**Validation:** Repository and deployment inventory.

---

### ASM-A3: Synchronous customer journey

**Assumption:** Primary browse and checkout journeys are **request-response synchronous** from the customer's perspective, aside from payment provider redirect.

**Source:** Typical web storefront pattern.

**Impact if wrong:** If heavy async or batch behaviour exists (e.g., quote generation, B2B workflows), latency and failure NFRs need revision.

**Validation:** Journey mapping with engineering walkthrough.

---

## Integration and Data Assumptions

### ASM-I1: Existing backend dependencies

**Assumption:** Product catalogue, inventory, orders, and payment remain integrated via **existing APIs or services** that may or may not migrate in this phase.

**Source:** Not stated — assumed partial migration.

**Impact if wrong:** If all backends must migrate simultaneously, programme scope, timeline, and risk multiply. If APIs are legacy and fragile, cloud migration does not fix integration reliability.

**Validation:** Integration inventory with ownership and SLA per dependency.

---

### ASM-I2: Payment provider stability

**Assumption:** The authorised payment provider remains unchanged through migration; redirect or hosted payment model continues.

**Source:** Not stated.

**Impact if wrong:** Payment provider change during migration compounds risk. PCI and security boundary NFRs require rework.

**Validation:** Commercial contract and technical integration documentation.

---

### ASM-I3: Inventory freshness tolerance

**Assumption:** Product availability shown to customers may be **eventually consistent** with up to **5 minutes staleness** except at checkout validation where real-time check occurs.

**Source:** Engineering estimate — common pattern, unconfirmed.

**Impact if wrong:** If marketing requires real-time stock during flash sales, caching assumptions and oversell risk controls change.

**Validation:** Business rule for oversold inventory during campaigns; historical oversell incidents.

---

## Organisational Assumptions

### ASM-O1: Engineering team size

**Assumption:** **2–5 engineers** responsible for build and operate, without dedicated SRE or platform team.

**Source:** "Small engineering team" — exact headcount unknown.

**Impact if wrong:** If team is solo or larger with platform support, operability NFR-O1 constraint shifts.

**Validation:** Org chart and on-call roster.

---

### ASM-O2: Cloud experience

**Assumption:** Team has **limited production experience** operating cloud infrastructure at this scale — migration includes learning curve.

**Source:** Inferred from migration initiative context.

**Impact if wrong:** If team is already cloud-native, operational simplicity constraint may be relaxed in favour of optimisation.

**Validation:** Skills assessment; prior cloud production ownership.

---

### ASM-O3: On-call during sales

**Assumption:** At least **one engineer** can be available within **30 minutes** during advertised peak sales events — not necessarily 24/7 year-round.

**Source:** Inferred from small team + high availability statement.

**Impact if wrong:** If no human response during overnight sale launch, Tier 1 availability claims require automation-only recovery — higher bar.

**Validation:** Marketing event schedule vs team availability policy.

---

### ASM-O4: Migration timeline

**Assumption:** Migration target for customer-facing storefront is **3–6 months** to production cutover before next major seasonal sale.

**Source:** Not stated — engineering estimate for planning.

**Impact if wrong:** Compressed timeline forces lift-and-shift without validation; extended timeline changes cost of dual-running environments.

**Validation:** Executive deadline tied to named commercial event.

---

## Business and Compliance Assumptions

### ASM-B1: UK-only storefront

**Assumption:** Primary customers are UK-based; GBP pricing; UK VAT display; no EU OSS complexity required at launch.

**Source:** "Small UK e-commerce company."

**Impact if wrong:** Multi-region latency, tax, and data residency requirements expand.

**Validation:** Shipping destinations; revenue by country.

---

### ASM-B2: Regulatory scope

**Assumption:** Compliance scope is **UK GDPR / DPA 2018** and standard consumer trading obligations — **not** sector-specific regulation (FCA, HIPAA, etc.).

**Source:** Stakeholder brief.

**Impact if wrong:** Additional controls may be mandatory regardless of architecture preference.

**Validation:** Legal/compliance confirmation; payment SAQ scope.

---

### ASM-B3: High availability interpretation

**Assumption:** "High availability" means **no prolonged complete outage during trading hours** — not mathematical five-nines on every component.

**Source:** Interpretation pending validation.

**Impact if wrong:** If business expects near-zero downtime during multi-hour sales, cost and complexity constraints conflict — must be resolved before architecture.

**Validation:** Executive statement of downtime tolerance in minutes during named sale.

---

## Financial Assumptions

### ASM-F1: Budget envelope

**Assumption:** Monthly infrastructure run cost target is **not yet defined numerically** but must not exceed a ** modest multiple of current hosting spend** without finance approval.

**Source:** "Budget-conscious" — no figures provided.

**Impact if wrong:** Architecture optimising for minimum cost vs maximum availability cannot be balanced without a number.

**Validation:** Current monthly hosting invoice; approved cloud budget range for year one.

---

### ASM-F2: Migration budget

**Assumption:** One-time migration effort is funded within **existing engineering capacity** — no large external consultancy budget.

**Source:** Inferred from team size and budget consciousness.

**Impact if wrong:** If consultancy available, timeline and risk profile change.

**Validation:** Programme budget and vendor engagement policy.

---

## Assumption Register Summary

| ID | Assumption | Confidence | Priority to validate |
|----|------------|------------|----------------------|
| ASM-T2 | 5–10× peak multiplier | Low | **Critical** |
| ASM-T3 | Concurrent user estimates | Low | **Critical** |
| ASM-F1 | Budget envelope undefined | Low | **Critical** |
| ASM-B3 | HA meaning | Low | **Critical** |
| ASM-I1 | Backend migration scope | Low | **Critical** |
| ASM-O4 | Migration timeline | Low | High |
| ASM-S3 / NFR-S3 | Payment boundary | Low | High |
| ASM-T1 | 5k daily users | Medium | Medium |
| ASM-A1 | Stateless tier | Medium | Medium |
| ASM-O1 | Team 2–5 engineers | Medium | Medium |
| ASM-B1 | UK-only | Medium | Medium |
| ASM-B2 | GDPR scope only | Medium | Low |

---

## Related Documents

- [03 — Non-functional Requirements](03-non-functional-requirements.md)
- [05 — Constraints](05-constraints.md)
- [07 — Architecture Discovery Summary](07-architecture-discovery-summary.md)
