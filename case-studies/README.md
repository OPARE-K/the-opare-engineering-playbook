# Case Studies

Case studies are end-to-end architectural narratives — real scenarios documented with the depth expected in a senior engineer's portfolio or an architecture review board presentation. They demonstrate the [Opare Method](../README.md#the-opare-method) and [engineering principles](../engineering-principles/README.md) applied to concrete problems.

This directory is intentionally empty in Phase 0. The structure and template are ready; content will be added in later phases.

---

## What a Case Study Is

A case study is **not**:

- A tutorial on how to click through the AWS console
- A Terraform module showcase without reasoning
- A generic "best practices" blog post with a diagram

A case study **is**:

- A documented answer to a business problem with explicit constraints
- An architecture with justified service choices and rejected alternatives
- A failure analysis that shows what breaks and how the system responds
- A cost model that reflects reality at launch and at scale
- A record of lessons learned — including mistakes

---

## What Each Case Study Will Include

Future case studies will follow [`case-study-template.md`](case-study-template.md) and cover:

| Section | Purpose |
|---------|---------|
| Business problem | Why the system exists |
| Requirements | Functional and non-functional |
| Architecture overview | Components and boundaries |
| Traffic flow | How requests and data move |
| AWS services and justification | Why these services, not others |
| Alternatives considered | Trade-off transparency |
| Security, networking, IAM | Defence in depth |
| Reliability and failure analysis | What fails, blast radius, recovery |
| Observability | SLIs, logging, alerting |
| Cost analysis | Run cost and drivers |
| Terraform structure | IaC organisation (when applicable) |
| Testing approach | How confidence is built |
| Interview questions | How to discuss the design under scrutiny |
| Lessons learned | Honest retrospective |

---

## How Case Studies Will Be Written

1. **Start with the problem** — not the solution. The reader should understand the business need before seeing any architecture.
2. **Apply the Opare Method** — all eight questions must be answerable from the document.
3. **Run the review framework** — complete the [architecture review checklist](../architecture-review-framework/README.md) before finalising.
4. **Record ADRs** — significant decisions get linked ADRs, not inline hand-waving.
5. **Include diagrams** — architecture and traffic flow; keep them readable, not decorative.
6. **Be honest about limits** — what was deferred, what would break at 10x scale, what you'd do differently.

---

## Naming Convention

```
case-studies/
├── README.md
├── case-study-template.md
├── 01-event-driven-order-processing/
│   └── README.md
├── 02-multi-tenant-saas-platform/
│   └── README.md
└── ...
```

Each case study lives in its own numbered directory. The main narrative is in `README.md` within that directory. Supporting artifacts (diagrams, ADR links) are referenced from there.

---

## Status

**Phase 0** — Template only. No case studies published yet.

Use [`case-study-template.md`](case-study-template.md) when authoring the first case study.
