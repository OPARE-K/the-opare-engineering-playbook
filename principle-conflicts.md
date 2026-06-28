# Principle Conflicts

Engineering principles do not always align. A design that maximises one principle often deprioritises another. The mistake is not choosing — it is choosing without documenting what was traded away.

When principles conflict, record the resolution in an ADR. Reference this guide for common patterns.

See also [Engineering Principles](engineering-principles/README.md) and [Manifesto](manifesto.md).

---

## Security vs Simplicity

**The tension:** Strong security adds layers — mTLS, fine-grained IAM, encryption everywhere, network segmentation. Each layer adds configuration, failure modes, and cognitive load.

### When security should win

- Sensitive data — PII, payment, credentials, health records
- Public or untrusted network exposure
- Regulatory or contractual obligation
- High blast radius of compromise

### When simplicity should win

- Internal-only, low-sensitivity tooling with trusted actors
- Early prototype with explicit sunset and no production data
- Team lacks capacity to operate complex security stack correctly — **misconfigured security is worse than simpler adequate controls**

### What must be documented

- Data classification and threat model scope
- Controls applied and controls deliberately omitted
- Residual risk and owner
- Trigger to increase security (e.g., external exposure, data classification upgrade)

---

## Reliability vs Cost

**The tension:** Multi-AZ, replication, warm standby, and redundant dependencies increase bill and complexity. Single-AZ and best-effort recovery are cheaper — until the outage.

### When reliability should win

- Tier 1 customer-facing revenue paths — see [Architecture Tiers](architecture-tiers.md)
- Data that cannot be recreated — RPO near zero
- Regulatory uptime requirements
- Dependency of other Tier 1 systems on this service

### When cost should win

- Batch, analytics, or internal tooling with tolerant users
- Explicitly documented RTO/RPO allowing hours of recovery
- Early stage where survival depends on burn rate
- Non-critical path where failure degrades but does not halt the business

### What must be documented

- Tier assignment and availability target
- Single points of failure accepted
- Estimated cost of redundancy vs estimated cost of downtime
- Recovery procedure and tested restore evidence (or accepted gap)

---

## Scalability vs Complexity

**The tension:** Sharding, event-driven architectures, CQRS, and microservices scale — but multiply operational surfaces, debugging difficulty, and deployment coordination.

### When scalability should win

- Proven or highly probable need to scale beyond single-node or single-region limits
- Traffic or data growth on critical path within planning horizon
- Cost of redesign mid-flight exceeds cost of upfront complexity

### When simplicity (lower complexity) should win

- Current and next-order-of-magnitude load fits monolith or managed service
- Team size cannot operate distributed system safely
- Premature scale optimisation — "we might need Kafka someday"

### What must be documented

- Load assumptions and revisit trigger (e.g., 10k RPS, 1TB data)
- Scaling path for v2 without claiming v1 must implement it
- Known bottleneck and acceptable timeline before address

---

## Performance vs Maintainability

**The tension:** Performance optimisation — custom caching, hand-tuned queries, bare-metal tuning, exotic data structures — often reduces readability and portability.

### When performance should win

- Latency or throughput is a contractual or competitive requirement
- Identified bottleneck with measured user or cost impact
- Hot path where optimisation is bounded and tested

### When maintainability should win

- Performance meets NFR with headroom
- Optimisation is speculative — no profile data
- Team turnover risk — clever code without owners

### What must be documented

- Measured baseline and target — not assumed
- Scope of optimisation — which paths, which SLIs
- Test strategy to prevent regression
- Plan to simplify if requirements relax

---

## Automation vs Flexibility

**The tension:** Automation — pipelines, policy-as-code, enforced guardrails — reduces toil and drift. It also constrains ad hoc changes and slows exception handling.

### When automation should win

- Repeated operations with failure cost — deploy, backup, scaling
- Compliance requires auditable, reproducible change
- Team toil exceeds capacity for improvement work
- Multiple environments must stay consistent

### When flexibility should win

- Early exploration — patterns not yet stable
- Genuine one-off migration with hard deadline
- Automation cost exceeds manual cost for low-frequency operation — **temporary** manual process with sunset

### What must be documented

- What is automated and what remains manual
- Exception process and who can approve break-glass
- Plan to automate manual steps if frequency increases

---

## Observability vs Cost

**The tension:** Comprehensive metrics, long log retention, distributed tracing, and third-party APM improve debuggability — at direct infrastructure cost and cardinality risk.

### When observability should win

- Tier 1 services with on-call accountability
- Complex distributed paths where incidents are hard to diagnose
- Compliance audit trail requirements
- Post-incident MTTR exceeded due to missing signals

### When cost should win

- Low-tier internal batch with email-on-failure sufficient
- Retention beyond compliance need without query use case
- High-cardinality metrics that dominate bill without proportional value

### What must be documented

- SLIs required and signals chosen to support them
- Retention periods and rationale
- What is **not** measured and accepted blind spots
- Cost estimate for observability stack at launch scale

---

## General Rule

When two principles conflict:

1. Name both principles explicitly.
2. State which won and **why in this context**.
3. Record what was deprioritised and the risk accepted.
4. Define trigger to revisit — incident, scale, compliance change.
5. Link to ADR.

Principle conflict is not design failure. Undocumented conflict is.

---

## Related

- [Decision Matrix](decision-matrix.md)
- [Architecture Tiers](architecture-tiers.md)
- [ADR Template](architecture-decision-records/adr-template.md)
