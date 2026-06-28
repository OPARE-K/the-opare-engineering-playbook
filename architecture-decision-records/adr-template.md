# ADR-NNNN: [Short Title of Decision]

> Copy this template to `NNNN-short-title-in-kebab-case.md` (e.g., `0003-adopt-dynamodb-for-session-store.md`). Replace placeholders and delete instructional comments.

## Status

**Proposed** | Accepted | Deprecated | Superseded by [ADR-XXXX](XXXX-link.md)

**Date:** YYYY-MM-DD

**Deciders:** [Names or roles — e.g., Platform Team, Principal Architect]

---

## Context

What is the issue or situation driving this decision?

- Business or technical background
- Constraints — timeline, budget, compliance, team skills
- Forces at play — scale, reliability targets, existing systems
- Why a decision is needed now

*Example: User session state is currently stored in-memory on application servers. Horizontal scaling requires sticky sessions, which complicates deployments and reduces availability during rollouts.*

---

## Decision

What is the change we are proposing or have agreed to implement?

State the decision clearly and unambiguously. One or two paragraphs.

*Example: We will store session data in Amazon DynamoDB with a TTL attribute, accessed via IAM roles from ECS tasks. Application servers will be stateless.*

---

## Alternatives Considered

| Option | Summary | Why not chosen |
|--------|---------|----------------|
| **Option A** — [Name] | [Brief description] | [Reason rejected] |
| **Option B** — [Name] | [Brief description] | [Reason rejected] |
| **Option C** — [Name] | [Brief description] | [Reason rejected] |

Include at least two genuine alternatives. "Do nothing" counts if it was seriously evaluated.

---

## Consequences

### Positive

- [Benefit 1]
- [Benefit 2]

### Negative

- [Drawback 1]
- [Drawback 2]

### Neutral

- [Side effect that is neither clearly good nor bad]

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| [Risk description] | Low / Med / High | Low / Med / High | [How we address or accept] |

---

## Cost Implications

- **Build cost:** [Engineering effort, migration, one-time AWS setup]
- **Run cost:** [Estimated monthly at launch and at projected scale]
- **Change cost:** [Cost to reverse or evolve this decision later]

*Include order-of-magnitude figures where possible. Link to Cost Explorer estimates or case study cost section if available.*

---

## Security Implications

- **Data classification:** [What data is affected]
- **Access model:** [IAM, encryption, network boundaries]
- **New attack surfaces:** [If any]
- **Compliance impact:** [If any]

*Reference [Security by Design](../engineering-principles/security-by-design.md) principles where relevant.*

---

## References

- [Link to architecture diagram]
- [Link to review checklist or meeting notes]
- [AWS documentation, RFCs, related ADRs]
- [Related case study](../case-studies/)

---

## Review Date

**Scheduled review:** YYYY-MM-DD (typically 30–90 days post-implementation)

**Last reviewed:** YYYY-MM-DD | Not yet reviewed

Post-implementation review compares design intent to production reality. See [Engineering Workflow](../engineering-workflow.md).

---

## Production Findings

Updated at review date — leave blank at acceptance if not yet in production.

| Finding | Expected | Actual | Action |
|---------|----------|--------|--------|
| [e.g., Monthly run cost] | [Estimate] | [Observed] | [Revise, accept, optimise] |
| [e.g., Failure mode under dependency timeout] | [Degrade gracefully] | [What happened] | [ADR supersession, fix, runbook] |

---

## Lessons Learned

What production, incidents, or operation taught that the original decision did not anticipate.

- **What worked as expected:**
- **What did not:**
- **What we would do differently:**

---

## Would We Make This Decision Again?

**Yes / Yes with changes / No**

Brief rationale. Honest assessment — not justification for sunk cost. If **No**, link to superseding ADR or planned revision.

---

## Evolution Path

How this decision is expected to change over time.

- **Revisit triggers:** [e.g., traffic exceeds 10k RPS, team doubles, compliance scope expands]
- **Planned v2 direction:** [If known]
- **Superseded by:** [ADR-XXXX when applicable]
- **One-way vs two-way door:** [Reversibility assessment]

---

## Notes

Optional: follow-up actions, triggers to revisit before scheduled review.

- [ ] [Action item]
