# ER-001: Constraints

**Engineering Review:** ER-001 — Highly Available Stateless Web Platform  
**Document:** 05 — Constraints  
**Status:** Discovery  
**Date:** 2026-06-28

---

## Purpose

Constraints narrow the solution space. Unlike assumptions — which may be true or false — constraints are **conditions the architecture must accept** at discovery time.

Each constraint explains **how it will influence future architecture decisions** without prescribing solutions.

---

## Organisational Constraints

### CON-O1: Small engineering team

**Constraint:** A small team (assumed 2–5 engineers) builds, migrates, and operates the platform. No dedicated platform engineering or SRE function is available.

**Influence on future architecture:**

- Favours designs with **lower operational surface area** — fewer independently operated components, fewer bespoke integrations to maintain.
- Limits acceptable complexity of deployment pipelines, service mesh, and multi-region active-active patterns unless automation is nearly zero-touch.
- On-call load must remain survivable — alert noise and operational toil directly threaten sustainability.
- Knowledge concentration risk: architecture must be **documentable and operable** by engineers who were not original authors.

**Trade-off preview:** Reliability vs complexity and operability — see [Principle Conflicts](../../principle-conflicts.md).

---

### CON-O2: Limited cloud operations maturity (assumed)

**Constraint:** Team is migrating from non-cloud or early cloud usage — production cloud operations at target scale are not yet established practice.

**Influence on future architecture:**

- Migration path must include **operational runway** — runbooks, training, and phased cutover — not big-bang replatforming without rollback.
- Designs requiring expert tuning (kernel, network, exotic database administration) are misaligned unless managed service absorbs that burden.
- Post-implementation review and ADR feedback loops are mandatory programme elements, not optional.

---

### CON-O3: On-call capacity is finite

**Constraint:** Extended 24/7/on-call coverage year-round is not assumed available. Peak sale coverage may be negotiated but is not unlimited.

**Influence on future architecture:**

- High availability targets must align with **[Architecture Tiers](../../architecture-tiers.md)** and actual paging capacity — claiming Tier 1 without Tier 1 on-call is an undocumented principle conflict.
- Self-healing, automated rollback, and clear degradation behaviour reduce dependence on human response at 2am.
- Incident detection must be **actionable** — small teams cannot investigate ambiguous alerts under pressure.

---

## Financial Constraints

### CON-F1: Budget-conscious organisation

**Constraint:** Infrastructure spend is scrutinised. Cost must be predictable, attributable, and justified against revenue — especially outside peak trading periods.

**Influence on future architecture:**

- Strong pressure against **year-round peak provisioning** — elasticity vs always-on redundancy becomes a central trade-off.
- NAT-equivalent data transfer, idle resources, and observability retention costs must be modelled explicitly in ADRs — not discovered post-launch.
- Tier 1 reliability features (multi-site redundancy, warm DR) require **explicit finance sign-off** tied to revenue-at-risk analysis.
- Tagging and cost allocation are architectural requirements, not FinOps afterthoughts.

---

### CON-F2: Undefined but bounded budget envelope

**Constraint:** No numeric cloud budget approved at discovery — architecture must be developed in **phases with cost gates** rather than unlimited design-first.

**Influence on future architecture:**

- Deliver **order-of-magnitude cost estimates** at proposal stage for normal month vs peak month vs migration month.
- Prefer reversible decisions (two-way doors) until budget confirms scale of investment.
- Option to phase: migrate browse path before checkout, or non-prod first — if programme constraint requires it.

---

## Technical Constraints

### CON-T1: Stateless application tier

**Constraint:** Customer-facing compute must not rely on instance-local durable state for correctness.

**Influence on future architecture:**

- Session/basket state must externalise or tokenise — influences state store selection, consistency, and failure behaviour.
- Rolling deployments and auto-replacement of failed instances become viable — supports NFR-R3.
- Any discovered hidden state (uploads, local cache, sticky sessions) becomes migration blocker until remediated.

---

### CON-T2: Existing integrations must continue to function

**Constraint:** Migration must not break contracts with payment, inventory, order, email, and identity dependencies without coordinated change programmes.

**Influence on future architecture:**

- Egress connectivity, API compatibility, timeout behaviour, and IP allowlisting from partners may constrain network and cutover design.
- Dual-running period likely required — migration architecture must tolerate **parallel old and new** environments temporarily (cost impact).
- Integration failure modes dominate customer-visible outages — dependency classification is P0 review work.

---

### CON-T3: High availability is mandatory for revenue path

**Constraint:** Business requires high availability for customer-facing storefront — interpreted as no acceptable **prolonged complete outage** during trading, with stricter expectation during sales.

**Influence on future architecture:**

- Minimum redundancy and health-routing behaviour required for serving tier — extent negotiated via tier definition and budget.
- Single points of failure in checkout path require explicit acceptance or mitigation before launch.
- DR and backup requirements flow from RTO/RPO NFRs — cannot be deferred as "phase 2" without documented risk acceptance.

---

### CON-T4: Seasonal traffic shape

**Constraint:** Traffic is uneven — low baseline, sharp peaks — not flat high volume.

**Influence on future architecture:**

- Capacity model must address **peak hour of peak day**, not daily average users alone.
- Load testing scenarios must include spike behaviour, not steady-state only.
- Cost architecture must handle **10× burst** without 10× annual bill unless business accepts that trade-off.

---

## Geographic and Market Constraints

### CON-G1: UK customer base

**Constraint:** Primary users located in the United Kingdom.

**Influence on future architecture:**

- Latency optimisation should prioritise UK end-user experience — influences region/location decisions when they arise.
- Data protection expectations align with UK GDPR — influences logging, retention, subprocessors, and DPA documentation.
- Trading hours and seasonal events follow UK retail calendar — influences change freeze and on-call planning.

---

## Regulatory and Compliance Constraints

### CON-R1: UK GDPR and Data Protection Act 2018

**Constraint:** Personal data processing must comply with UK data protection law — lawful basis, retention limits, subject rights, breach notification readiness.

**Influence on future architecture:**

- Data minimisation in logs and traces; PII redaction in observability stack.
- Subprocessor and data processing agreements for any third-party hosted components.
- Data residency preferences may apply — UK or adequate jurisdiction — pending legal input, not assumed EU-only.

---

### CON-R2: No sector-specific regulatory regime beyond standard UK business practice

**Constraint:** No FCA, HIPAA, PCI-DSS programme-level audit, or public sector classification claimed at discovery — **payment scope excepted**.

**Influence on future architecture:**

- Avoid over-building compliance controls that consume budget and operability without legal mandate.
- **Payment provider boundary remains critical** — even without full PCI programme, card data handling rules constrain design.
- Constraint may change if legal review identifies PCI SAQ obligation — triggers architecture revisit.

---

### CON-R3: Consumer trading expectations

**Constraint:** UK Consumer Rights Act and e-commerce regulations imply orders once confirmed must be honourable; system failures during checkout have commercial and dispute consequences.

**Influence on future architecture:**

- Order idempotency and auditable order state transitions are functional reliability requirements.
- Extended checkout outage during paid-advertising windows has measurable commercial liability beyond infrastructure cost.

---

## Programme and Timeline Constraints

### CON-P1: Migration before next major sale (assumed)

**Constraint:** Business value is tied to readiness for next seasonal peak — slipping past that date materially reduces programme ROI.

**Influence on future architecture:**

- Scope must be **cuttable** — MVP migration path vs full optimisation path.
- Validation time (load test, DR test, game day) must be scheduled in programme plan, not squeezed post-build.
- Risk acceptance process must be fast — small team cannot absorb long review cycles without missing window.

---

### CON-P2: Minimal customer-visible disruption on cutover

**Constraint:** Hard cutover with prolonged maintenance window is commercially unacceptable during trading periods.

**Influence on future architecture:**

- Cutover strategy must support **low-downtime or zero-downtime migration** for storefront — influences DNS, session handling, and dual-run approach.
- Maintenance windows limited to low-traffic periods — typically overnight UK, avoiding Q4 peak.

---

## Explicit Non-Constraints (Freedom Reserved)

The following are **not** constrained at discovery — architecture may propose options freely:

- Specific cloud provider services (cloud migration direction stated but not designed here)
- Monolith vs decomposed services beyond ASM-A2
- Container vs VM vs platform-managed compute
- Specific database or cache technology
- Multi-region vs single-region (subject to NFRs and CON-F1)

---

## Constraint Interaction Matrix

| Constraint pair | Tension | Resolution venue |
|-----------------|---------|------------------|
| CON-F1 vs CON-T3 | Cost vs high availability | Tier definition + finance sign-off |
| CON-O1 vs CON-T3 | Team size vs redundancy | Simplicity vs reliability trade-off |
| CON-O3 vs CON-T3 | On-call vs availability target | Architecture tiers + accepted risk |
| CON-P1 vs CON-O2 | Timeline vs operational maturity | Phased migration scope |
| CON-T4 vs CON-F1 | Peak scale vs budget | Elasticity vs over-provision decision |

---

## Related Documents

- [04 — Assumptions](04-assumptions.md)
- [06 — Success Criteria](06-success-criteria.md)
- [Organisational Constraints](../../organisational-constraints.md)
- [Architecture Tiers](../../architecture-tiers.md)
