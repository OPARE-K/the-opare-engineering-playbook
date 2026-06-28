# Scalability

## What This Principle Means

Scalability is the ability of a system to handle increased load — in requests, data volume, users, or geographic reach — without proportional degradation in performance or disproportionate increase in operational burden. It is not simply "can we add more servers." It is whether the **architecture** allows growth along the dimensions that matter for the workload.

Scalability has two common forms:

- **Vertical scaling** — increase capacity of existing resources (larger instance, more CPU/RAM)
- **Horizontal scaling** — add more instances, shards, or partitions

Cloud favours horizontal scaling where the application and data model permit it. The principle is to design so that scaling is a **configuration change**, not a redesign.

---

## Why It Matters in Cloud Architecture

Demand is rarely static. Product launches, seasonal peaks, viral events, and organic growth all stress systems that were sized for yesterday's traffic. Architectures that scale poorly become bottlenecks — on a single database, a synchronous choke point, or a stateful component that cannot be replicated.

Poor scalability also drives cost: over-provisioning "just in case" wastes money; under-provisioning causes outages and emergency firefighting. The goal is **elasticity** — capacity that tracks demand within defined bounds.

Scalability intersects with reliability. A system that cannot scale under load will fail in ways that look like reliability problems but are actually capacity problems.

---

## Design Questions to Ask

- What are the **scaling dimensions** — read vs write, compute vs storage, synchronous vs async?
- Where is the **bottleneck** likely to appear first under 2x, 10x, 100x load?
- Is the application **stateless** at the compute layer? Where is state held, and can it be partitioned?
- Can the **data layer** scale — read replicas, sharding, caching, purpose-built stores (DynamoDB, ElastiCache)?
- What is **cached**, and what is the cache invalidation strategy?
- Are there **synchronous dependencies** that do not scale with the same curve as the caller?
- How does **auto-scaling** trigger — CPU, queue depth, custom metrics, schedule?
- What are **scaling limits** of chosen services (API rate limits, connection limits, account quotas)?
- Is the system **multi-region** or multi-AZ by design, or is that a future phase?
- What load has been **tested** — load tests, soak tests, spike tests?
- Does scaling **increase cost linearly**, super-linearly, or step-wise?

---

## Common Mistakes to Avoid

| Mistake | Why it hurts |
|---------|--------------|
| Scaling only the web tier while the database is fixed | Classic bottleneck; app scales, DB melts |
| Treating the database as infinite | Connection pools, IOPS, and lock contention have limits |
| No caching strategy for hot read paths | Repeated identical work under load |
| Stateful sessions tied to individual instances | Horizontal scaling requires sticky sessions or external session store |
| Synchronous fan-out to N services per request | Latency and failure multiply with scale |
| Ignoring AWS service quotas and rate limits | Sudden growth hits hard limits with no automatic relief |
| Auto-scaling on the wrong metric | Scales too late (CPU already saturated) or too early (cost spike) |
| Premature microservices for "scalability" | Operational overhead without proven need |
| No load testing before launch | First real spike is the test |
| Assuming serverless scales infinitely without cold starts or concurrency limits | Lambda and similar services have boundaries |

---

## In Practice

Scale for the **next order of magnitude**, not the next decade. Document known limits and the path to the next tier (e.g., "at 10k RPS we will introduce read replicas; at 50k RPS we evaluate sharding").

Not every component needs to scale the same way. A batch analytics pipeline and a real-time API have different scaling profiles. Match the pattern to the workload — and document when you are consciously deferring scale (with triggers for when to revisit).
