# Reliability

## What This Principle Means

Reliability is the ability of a system to perform its required function under stated conditions for a stated period of time. In cloud architecture, this means designing for **partial failure** — because components will fail, networks will partition, and dependencies will degrade — while maintaining acceptable service for users and downstream systems.

Reliability is not the same as availability. A system can be "available" but unreliable if it returns incorrect data, drops requests silently, or requires manual intervention to recover. Reliability encompasses correctness, consistency expectations, recovery time, and predictable behaviour under stress.

---

## Why It Matters in Cloud Architecture

Cloud platforms abstract hardware failure, but they do not eliminate it. EC2 instances terminate. AZs have outages. API rate limits throttle you. Third-party SaaS dependencies fail on their own schedules.

Architectures that assume perfect infrastructure fail loudly and expensively in production. Architectures that assume failure and design for containment, detection, and recovery fail gracefully — or fail in ways that are understood and bounded.

Reliability directly affects:

- **Customer trust** — outages and data loss have lasting reputational cost
- **Operational load** — unreliable systems generate pager noise and firefighting
- **Cost** — redundancy and resilience have a price; so does downtime
- **Compliance** — many frameworks require defined RTO/RPO and incident response

---

## Design Questions to Ask

- What is the **blast radius** if this component fails? One user, one tenant, one region, everything?
- What are the **dependencies** of this service, and what happens when each one is unavailable or slow?
- What is the acceptable **Recovery Time Objective (RTO)** and **Recovery Point Objective (RPO)** for this workload?
- Is this path **synchronous or asynchronous**? What happens to in-flight work on failure?
- Where are **single points of failure**? Are they acceptable, mitigated, or documented as accepted risk?
- How does the system behave under **degradation** — can it shed load, serve stale data, or queue work?
- What **health checks** exist, and do they reflect real readiness (not just process liveness)?
- How will we **detect** failure before users report it?
- What is the **runbook** for the most likely failure scenarios?
- Have we tested failure — **chaos engineering, game days, failover drills**?

---

## Common Mistakes to Avoid

| Mistake | Why it hurts |
|---------|--------------|
| Treating "multi-AZ" as sufficient without testing failover | Failover paths are often untested until the real event |
| No timeout or circuit breaker on external calls | One slow dependency stalls the entire request path |
| Synchronous chains across unreliable boundaries | Latency and failure compound multiplicatively |
| Health checks that only verify the process is running | Traffic routed to instances that cannot serve |
| Ignoring dependency SLOs | Your reliability ceiling is your weakest dependency |
| Over-engineering for five-nines when the business needs three | Cost and complexity without proportional value |
| No idempotency on retried operations | Retries cause duplicate charges, duplicate records, duplicate side effects |
| Backups without restore testing | Backups that cannot be restored are wishful thinking |
| Assuming auto-scaling fixes reliability | Scaling adds capacity; it does not fix logic bugs or data corruption |

---

## In Practice

Reliability is a spectrum negotiated with the business, not a binary. Document the target (e.g., 99.9% monthly availability, RPO of 1 hour, RTO of 4 hours) and design explicitly to meet it — or document why you are accepting less and what the business impact is.

When reliability conflicts with cost or speed to market, record the trade-off in an ADR. "We chose single-AZ for this non-critical batch workload" is a valid decision if it is conscious and documented.
