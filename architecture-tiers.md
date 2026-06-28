# Architecture Tiers

Example service tier definitions for aligning reliability, operations, cost, and recovery expectations. **These are illustrative defaults** — calibrate to your business, contracts, and organisational capacity.

Tier assignment should be explicit in architecture review, ADRs, and case studies. Not every component in a system shares the same tier — a Tier 1 API may depend on a Tier 3 analytics pipeline.

See [Organisational Constraints](organisational-constraints.md) for on-call and team capacity alignment.

---

## Tier Summary

| | Tier 1 — Critical | Tier 2 — Important | Tier 3 — Standard |
|-|-------------------|--------------------|--------------------|
| **Profile** | Revenue, safety, regulatory core path | Significant user impact, recoverable | Internal, batch, best-effort |
| **Availability target (example)** | 99.95% (~22 min/month) | 99.9% (~43 min/month) | 99.5% (~3.6 hr/month) |
| **Example RPO** | 15 minutes | 1 hour | 24 hours |
| **Example RTO** | 1 hour | 4 hours | 24 hours |
| **Monitoring** | 24/7 paging on SLO breach | Business-hours paging or next-day response | Email/dashboard, no page |
| **Operations** | Dedicated on-call, runbooks, game days | Team on-call rotation, documented runbooks | Best-effort, business hours |
| **Cost expectation** | Highest — multi-AZ, redundancy, DR tested | Moderate — multi-AZ where justified | Lowest — single-AZ acceptable |
| **Recovery expectation** | Automatic failover where possible; tested DR | Documented manual recovery; tested restore | Rebuild from backup acceptable |

---

## Tier 1 — Critical

**Examples:** Payment processing, authentication, primary customer API, systems with regulatory uptime obligation.

### Availability target (example)

**99.95%** monthly — approximately 22 minutes unplanned downtime per month. Stricter targets require explicit business case and cost acceptance.

### Example RPO

**15 minutes** — maximum acceptable data loss. Implies frequent backup, replication, or durable write acknowledgment.

### Example RTO

**1 hour** — maximum time to restore acceptable service. Implies tested failover, warm standby, or rapid rebuild procedure.

### Monitoring expectation

- SLIs and SLOs defined with error budget
- 24/7 paging on user-visible symptom breach
- Distributed tracing on critical paths
- Synthetic canaries for primary journeys
- Dashboard accessible to on-call without tribal knowledge

### Operational expectation

- Named owner and on-call rotation
- Runbooks for top failure modes — linked from alerts
- Change management — canary or blue/green, tested rollback
- Post-incident review within 5 business days
- DR drill at least annually

### Cost expectation

- Multi-AZ as baseline for stateful and stateless compute
- Redundancy for dependencies where single failure causes outage
- Higher observability spend justified by MTTR reduction
- DR infrastructure cost included in estimate — not deferred silently

### Recovery expectation

- Automatic failover where architecture supports it
- Failover and restore **tested** — not assumed from diagram
- Communication plan for customer-facing impact
- Blameless postmortem for Sev1/Sev2 incidents

---

## Tier 2 — Important

**Examples:** Internal admin tools with wide usage, secondary product features, integration APIs with SLA to partners.

### Availability target (example)

**99.9%** monthly — approximately 43 minutes unplanned downtime per month.

### Example RPO

**1 hour** — acceptable for restorable data with periodic backup or replication.

### Example RTO

**4 hours** — manual recovery acceptable with documented procedure.

### Monitoring expectation

- SLOs defined; paging during business hours or extended coverage per team capacity
- Structured logs with correlation IDs
- Alerts on error rate and latency degradation
- Runbook links on primary alerts

### Operational expectation

- Team on-call rotation — may share rotation with Tier 1 team
- Runbooks for common incidents
- Rollback tested in staging
- Post-incident review for significant outages

### Cost expectation

- Multi-AZ for data stores where RPO/RTO require it; single-AZ acceptable for stateless where auto-replace suffices
- Balance redundancy cost against tier — do not apply Tier 1 cost to Tier 2 without justification

### Recovery expectation

- Documented restore procedure — tested at least once before launch
- Manual failover acceptable
- Degraded mode defined — what users see during partial failure

---

## Tier 3 — Standard

**Examples:** Batch reports, analytics pipelines, dev/staging environments, internal experiments with explicit sunset.

### Availability target (example)

**99.5%** monthly — approximately 3.6 hours unplanned downtime per month. Scheduled maintenance may be acceptable with notice.

### Example RPO

**24 hours** — daily backup sufficient for recreatable or non-critical data.

### Example RTO

**24 hours** — rebuild from backup or re-run batch acceptable.

### Monitoring expectation

- Dashboard and email alerts — no overnight paging unless promoted
- Log retention aligned with debug need, not maximum
- Owner notified on sustained failure

### Operational expectation

- Best-effort support during business hours
- Minimal runbook — rebuild documentation acceptable
- No on-call requirement unless promoted to Tier 2

### Cost expectation

- Single-AZ acceptable
- Spot or scheduled compute where interruption tolerable
- Minimal redundancy — cost optimised explicitly

### Recovery expectation

- Rebuild from IaC and backup
- Accept data loss within RPO
- Failure may wait until next business day

---

## Tier Assignment in Practice

1. Identify the **user-facing or business-critical path** — tier the path, not only individual components.
2. A Tier 1 path must not depend on a Tier 3 component without **documented degradation** or risk acceptance.
3. Record tier in ADR and architecture review.
4. Revisit tier when usage, revenue attribution, or compliance scope changes.

---

## Tier vs Principle Conflicts

Applying Tier 1 reliability to a Tier 3 workload wastes cost. Applying Tier 3 recovery to a Tier 1 path wastes trust.

When tier and stakeholder expectation disagree, escalate before build — not after outage.

See [Principle Conflicts](principle-conflicts.md) — reliability vs cost.

---

## Related

- [Organisational Constraints](organisational-constraints.md)
- [Reliability](engineering-principles/reliability.md)
- [Architecture Review Framework](architecture-review-framework/README.md)
- [ADR Template](architecture-decision-records/adr-template.md)
