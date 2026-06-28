# Architecture Decision Records

Architecture Decision Records (ADRs) capture **significant** technical decisions — the ones that are hard to reverse, affect multiple teams, or embody important trade-offs. They preserve context so future engineers understand not just *what* was decided, but *why*.

This playbook uses ADRs as the canonical record of architectural judgement. Not every choice needs an ADR. Use them when the decision:

- Affects system structure, non-functional properties, or dependencies
- Has meaningful alternatives that were evaluated
- Will be questioned in six months by someone who wasn't in the room
- Accepts risk or technical debt consciously

---

## When to Write an ADR

| Write an ADR | Skip an ADR |
|--------------|-------------|
| Choosing database technology | Naming a variable |
| Single-AZ vs multi-AZ for a tier | Routine dependency patch |
| Public vs private API exposure | Formatting preference |
| Event-driven vs synchronous integration | Bug fix with obvious correct answer |
| Multi-region strategy | Copying an established pattern unchanged |

When in doubt, write a short ADR. The cost of documentation is lower than the cost of re-debate.

---

## ADR Lifecycle

| Status | Meaning |
|--------|---------|
| **Proposed** | Under discussion; not yet adopted |
| **Accepted** | Decision is active and should be followed |
| **Deprecated** | Superseded or no longer recommended; see successor ADR |
| **Superseded** | Replaced by a newer ADR — link to replacement |

Store ADRs in this directory with sequential numbering:

```
architecture-decision-records/
├── README.md
├── adr-template.md
├── 0001-use-event-driven-integration.md
├── 0002-single-region-with-backup-dr.md
└── ...
```

---

## How to Write a Good ADR

1. **Start from context** — what forces shaped the decision? Business, technical, organisational.
2. **Name alternatives honestly** — including the option you rejected and why.
3. **Record consequences** — positive, negative, and neutral. Decisions have side effects.
4. **Include cost and security** — even rough estimates beat silence.
5. **Link references** — RFCs, AWS docs, prior incidents, review notes.
6. **Keep it concise** — one to two pages. Link to case studies for depth.

ADRs are immutable once accepted. If the decision changes, write a new ADR that supersedes the old one — do not rewrite history.

---

## Relationship to Other Artifacts

| Artifact | Relationship |
|----------|--------------|
| [Architecture Review Framework](../architecture-review-framework/README.md) | Reviews often produce ADR candidates |
| [Case Studies](../case-studies/) | Case studies may reference multiple ADRs |
| [Engineering Principles](../engineering-principles/README.md) | ADRs should cite principles applied or traded off |
| [Opare Method](../README.md#the-opare-method) | ADR sections map to method questions |

---

## Template

Use [`adr-template.md`](adr-template.md) for all new ADRs. Copy the template, rename with the next sequence number, and fill in each section.
