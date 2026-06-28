# Thinking Framework

The mental model used before making engineering decisions. Work through these questions in order — skipping early questions produces solutions looking for problems.

This framework complements the [Opare Method](README.md#the-opare-method) and applies to architecture proposals, ADRs, case studies, and reviews.

---

## Why Does This System Exist?

Before components, services, or diagrams: what is the purpose?

- What business or user outcome does this enable?
- What happens if we do not build it?
- Is this a new capability, a replacement, or an optimisation?

If you cannot state the purpose in two sentences, you are not ready to design.

---

## What Business Problem Are We Solving?

Separate the problem from the solution. "We need Kubernetes" is not a problem. "We need to deploy independently ten times per day with rollback under five minutes" is closer.

- What pain exists today?
- Who feels it?
- How is it measured?
- What does success look like in six months?

---

## Who Uses It?

Identify actors and their expectations:

- End users — latency, availability, experience under failure
- Internal teams — APIs, dashboards, integrations
- Administrators — configuration, access, audit
- Downstream systems — contracts, SLAs, data formats

Different users impose different non-functional requirements. Name them explicitly.

---

## Who Operates It?

The team that builds it is not always the team that runs it.

- Who is on-call?
- What skills does the operating team have?
- What tooling do they already use?
- Can they support this design at current team size?

A design that ignores operators becomes an organisational bottleneck.

See [Organisational Constraints](organisational-constraints.md).

---

## What Constraints Exist?

Constraints narrow the solution space. Document them before evaluating options.

- **Time** — launch deadline, regulatory date
- **Budget** — build and run cost ceilings
- **Compliance** — data residency, audit, certification
- **Technology** — existing stack, approved services, skills gap
- **Organisational** — team size, approval paths, support model
- **Dependencies** — systems you cannot change

Unstated constraints resurface as surprises in review.

---

## What Assumptions Are We Making?

Assumptions are decisions you have not yet validated. Make them visible.

| Assumption | If wrong |
|------------|----------|
| Traffic will not exceed 1k RPS in year one | Scaling redesign required |
| Third-party API is 99.9% available | Circuit breakers and fallbacks needed |
| Team of two can operate this stack | Platform support or simplification required |

Review assumptions at architecture review and revisit after launch.

---

## What Options Exist?

Generate at least two viable alternatives before selecting one. Include "do nothing" or "defer" if they were seriously considered.

For each option, sketch:

- High-level approach
- Primary benefits
- Primary costs and risks
- Reversibility

Use the [Decision Matrix](decision-matrix.md) for structured comparison when options are close.

---

## What Could Fail?

Failure is a design input, not a post-launch discovery.

- Component failures — compute, database, network, dependency
- Human failures — misconfiguration, bad deploy, wrong delete
- Scale failures — traffic spike, quota exhaustion, thundering herd
- Security failures — credential compromise, data exposure

Identify blast radius for each. See [Architecture Tiers](architecture-tiers.md) for recovery expectations by tier.

---

## What Happens When It Fails?

Designing for failure means defining behaviour, not assuming uptime.

- What does the user see?
- What data is lost or duplicated?
- What alerts fire?
- What is the recovery path — automatic, runbook, manual?
- What is the communication plan?

Untested failure paths are wishful thinking. Prefer documented degradation over silent breakage.

---

## What Does It Cost?

Cost has three horizons:

- **Build cost** — engineering time, migration, setup
- **Run cost** — monthly infrastructure and operational labour at launch and projected scale
- **Change cost** — effort to reverse, extend, or replace this decision

A cheap build with an expensive run model is a trap. A cheap run with an expensive change cost is lock-in.

---

## How Does It Evolve?

No design is final. Document the expected evolution path:

- What triggers a revisit — scale threshold, compliance change, team growth?
- What is the v2 direction if v1 succeeds?
- What is the deprecation path if v1 fails?
- Which decisions are two-way doors vs one-way doors?

Link forward to ADRs that may supersede this design. See [Engineering Workflow](engineering-workflow.md) for the full lifecycle.

---

## When to Use This Framework

| Situation | Apply |
|-----------|-------|
| New system or major capability | Full framework |
| Significant technology choice | Options, cost, failure, evolution |
| Pre-review self-assessment | All questions — flag gaps |
| Post-incident retrospective | Assumptions, failure, what happens when |

If a question cannot be answered, investigate before committing.

---

## Related

- [Manifesto](manifesto.md)
- [Engineering Workflow](engineering-workflow.md)
- [Decision Matrix](decision-matrix.md)
- [Architecture Review Framework](architecture-review-framework/README.md)
