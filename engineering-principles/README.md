# Engineering Principles

These principles are the foundation of every architectural decision documented in this playbook. They are not slogans — they are constraints and lenses applied during design, review, and operation.

When two principles conflict (and they will), the conflict itself should be documented as a trade-off in an ADR or case study.

---

## Principles

| Principle | Summary |
|-----------|---------|
| [Reliability](reliability.md) | Systems must behave predictably under failure and load |
| [Security by Design](security-by-design.md) | Security is architectural, not an afterthought |
| [Scalability](scalability.md) | Capacity must match demand without unnecessary complexity |
| [Cost Awareness](cost-awareness.md) | Every resource has a price; design with that in mind |
| [Observability](observability.md) | You cannot operate what you cannot see |
| [Operational Excellence](operational-excellence.md) | Running the system is as important as building it |

---

## How to Use These Principles

**During design.** Before selecting services or drawing boundaries, read the relevant principle files. Use the design questions in each document as a checklist.

**During review.** The [architecture review framework](../architecture-review-framework/README.md) maps directly to these principles. A design that violates a principle without documented justification should not proceed.

**During ADRs.** Reference which principles informed the decision and which were deprioritised.

**During case studies.** Each case study should demonstrate how principles were applied in a specific context — including where they conflicted and how that was resolved.

---

## Relationship to the Opare Method

The Opare Method asks *what* to decide. These principles define *how* to decide well:

- **Why does this exist?** → Grounded in business need, not technology preference
- **What principles matter?** → Answered by this directory
- **What could fail?** → Reliability, observability, operational excellence
- **What does it cost?** → Cost awareness
- **How does it evolve?** → Scalability, operational excellence

Read the individual principle files for depth. Return here when you need to align a team on shared vocabulary before a review.
