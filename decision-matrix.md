# Decision Matrix

A reusable template for comparing technology or architecture options. Use when two or more viable paths exist and the choice is not obvious from requirements alone.

**Scores are context-dependent.** There are no universal winners. A managed service may score high on operational simplicity for a small team and low on cost at scale. A self-hosted option may win on control and lose on time-to-market. The matrix forces explicit comparison — it does not replace judgement.

---

## How to Use

1. Name the decision and the options (minimum two).
2. Define context: team size, tier, timeline, constraints. See [Organisational Constraints](organisational-constraints.md).
3. Score each option per criterion using a consistent scale (e.g., 1–5 or Low / Med / High).
4. Add notes — a score without rationale is not useful.
5. Identify where scores conflict and apply [Principle Conflicts](principle-conflicts.md).
6. Record the outcome in an ADR.

Do not sum scores into a single number unless weights are explicitly agreed with stakeholders. Weighted totals hide trade-offs.

---

## Comparison Template

**Decision:** [What is being chosen]

**Context:** [Team, tier, timeline, key constraints]

**Date:** YYYY-MM-DD

| Criterion | Option A: [Name] | Option B: [Name] | Option C: [Name] |
|-----------|------------------|------------------|------------------|
| Business fit | | | |
| Simplicity | | | |
| Operational complexity | | | |
| Security | | | |
| Reliability | | | |
| Scalability | | | |
| Cost | | | |
| Observability | | | |
| Maintainability | | | |
| Change cost | | | |
| Vendor lock-in | | | |
| Future evolution | | | |

**Notes per option:**

### Option A — [Name]

- Strengths:
- Weaknesses:
- Best when:

### Option B — [Name]

- Strengths:
- Weaknesses:
- Best when:

---

## Criteria Definitions

### Business fit

Does this option deliver the required capability within timeline and organisational capability? Does it align with strategic direction?

### Simplicity

How understandable and operable is this for the team that will own it? Fewer moving parts, clearer failure modes, less bespoke glue code.

### Operational complexity

Day-two burden: patching, scaling, incident response, on-call load, required specialist skills.

### Security

Fit to data classification and threat model. Identity model, encryption, audit, blast radius of compromise.

### Reliability

Fit to tier availability target. Failure modes, recovery, dependency risk.

### Scalability

Fit to projected load growth. Bottlenecks, scaling path, service limits.

### Cost

Build, run, and change cost at launch and projected scale. Include operational labour where significant.

### Observability

Can the team measure SLIs, debug incidents, and correlate user impact with this option's telemetry model?

### Maintainability

Can the team upgrade, patch, and extend without rewriting? Documentation, community, vendor support lifecycle.

### Change cost

How hard to reverse or migrate away? Data portability, API coupling, contractual exit.

### Vendor lock-in

Dependency on proprietary APIs, data formats, or operational models that constrain future options.

### Future evolution

Does this option have a credible path to v2 — scale, region, feature, or team growth — without full redesign?

---

## Scoring Guidance

| Scale | Meaning |
|-------|---------|
| **1 / Low** | Poor fit for this context |
| **3 / Med** | Acceptable with documented trade-offs |
| **5 / High** | Strong fit for this context |

A low score is not a veto. It is a signal to document why the option was rejected — or why the trade-off was accepted.

---

## Example Context Statements

Use these to anchor scores — the same technology scores differently in different contexts:

- *"Option A wins on simplicity for a team of three with no dedicated platform engineer."*
- *"Option B wins on cost at 10x scale but loses on time-to-market for a Q3 launch."*
- *"Option C minimises vendor lock-in but increases operational complexity beyond current on-call capacity."*

---

## After the Matrix

1. Record decision in an [ADR](architecture-decision-records/adr-template.md).
2. Reference deprioritised criteria and [principle conflicts](principle-conflicts.md) if applicable.
3. Define **revisit trigger** — scale threshold, cost threshold, operational pain signal.

---

## Related

- [Thinking Framework](thinking-framework.md)
- [Principle Conflicts](principle-conflicts.md)
- [Organisational Constraints](organisational-constraints.md)
- [Architecture Tiers](architecture-tiers.md)
