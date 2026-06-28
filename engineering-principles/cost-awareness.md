# Cost Awareness

## What This Principle Means

Cost awareness means treating infrastructure spend as a **first-class design constraint** — not an afterthought discovered when the finance team asks why the AWS bill tripled. Every architectural choice has a cost profile: compute, storage, data transfer, licensing, operational labour, and the cost of change.

Cost awareness is not cost minimisation at all costs. It is **informed trade-off**: spending where it creates value, avoiding waste where it does not, and making spend **visible, attributable, and predictable**.

---

## Why It Matters in Cloud Architecture

Cloud billing is granular and dynamic. Resources left running, over-provisioned instances, unbounded data transfer, unused snapshots, and orphaned EBS volumes accumulate silently. Without design discipline, costs scale faster than revenue or value.

Cost also shapes **architecture viability**. A design that is technically correct but economically unsustainable will be cut, replaced, or operated under dangerous shortcuts (disabling backups, skipping multi-AZ) that undermine reliability and security.

FinOps and architecture are linked: tags, allocation, budgets, and anomaly detection only work when the architecture supports attribution — per team, per environment, per workload.

---

## Design Questions to Ask

- What is the **expected monthly run cost** at current scale, 2x scale, and 10x scale?
- Which components drive the **largest share** of spend — compute, storage, egress, managed services?
- Can we **attribute cost** to team, product, environment, or customer via tagging?
- Are we paying for **idle or over-provisioned** resources — right-sizing candidates?
- What is the **data transfer** path — cross-AZ, cross-region, internet egress? Egress is often the surprise line item.
- **Reserved capacity vs on-demand vs Spot** — what is the utilisation pattern and interruption tolerance?
- What is the **cost of redundancy** — multi-AZ, backups, DR — and is it aligned with RTO/RPO requirements?
- Are there **serverless vs always-on** trade-offs — Lambda vs ECS vs EC2 for this workload profile?
- What **storage tier** is appropriate — S3 Standard vs IA vs Glacier; gp3 vs io2?
- What is the **cost of change** — migration, refactor, or repatriation if the design proves too expensive?
- Are **budgets and alerts** configured before launch, not after the first bill shock?

---

## Common Mistakes to Avoid

| Mistake | Why it hurts |
|---------|--------------|
| No tagging strategy from day one | Cost cannot be allocated; optimisation is guesswork |
| Largest instance "to be safe" | Paying for capacity that is never used |
| Ignoring NAT Gateway and data transfer costs | Steady bleed on every cross-AZ and egress byte |
| Snapshots and AMIs without lifecycle policy | Storage costs grow unbounded |
| Dev/staging environments running 24/7 at production scale | Multiplier on waste |
| Choosing managed services without comparing unit economics | Convenience at premium without justification |
| No review cadence — set and forget | Drift, sprawl, and new services accumulate |
| Optimising compute while ignoring storage and egress | Fixing 20% of the bill while 50% leaks elsewhere |
| Cost as someone else's problem (finance, platform team only) | Engineers ship designs they cannot afford to operate |
| Cutting redundancy to save money without documenting accepted risk | Cheaper until the outage |

---

## In Practice

Include a **cost section** in every ADR and case study. Estimate order-of-magnitude monthly cost at launch and at projected scale. Revisit when architecture or traffic changes materially.

Cost awareness pairs with reliability and scalability: the cheapest design is rarely the most reliable; the most redundant is rarely the cheapest. Document where you landed on that spectrum and why.

Use AWS Cost Explorer, budgets, and tagging as operational tools — but design so those tools have meaningful data to work with.
