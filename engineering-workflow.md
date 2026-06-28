# Engineering Workflow

The lifecycle for every architecture documented in this repository. Not every stage requires a formal artifact every time — but skipping a stage without conscious justification is how gaps become incidents.

Map each stage to outputs: review checklist items, ADRs, threat models, runbooks, post-implementation notes.

---

## Lifecycle Overview

```
Problem Discovery
       ↓
Business Requirements
       ↓
Non-functional Requirements
       ↓
Architecture Proposal
       ↓
Architecture Review
       ↓
ADR
       ↓
Threat Model
       ↓
Implementation
       ↓
Validation
       ↓
Operations
       ↓
Post-Implementation Review
       ↓
Lessons Learned
       ↓
Architecture Revision
```

---

## Problem Discovery

**Why it exists:** Prevents building solutions without validated problems. Most wasted engineering effort starts here — a technology choice in search of a use case.

**Output:** Problem statement, stakeholders, success criteria, scope boundaries.

**Questions:** See [Thinking Framework](thinking-framework.md) — "why does this exist?" and "what business problem?"

---

## Business Requirements

**Why it exists:** Defines what the system must do for the business — capabilities, integrations, data flows, regulatory obligations. Without this, non-functional requirements float unattached to value.

**Output:** Functional requirements document or equivalent section in case study / ADR context.

**Gate:** Can you explain the business outcome without mentioning technology?

---

## Non-functional Requirements

**Why it exists:** Defines how the system must behave — availability, latency, throughput, RTO/RPO, retention, concurrency. NFRs drive architecture; functional requirements alone do not.

**Output:** NFR table with measurable targets. Align with [Architecture Tiers](architecture-tiers.md) where applicable.

**Gate:** Every NFR has a number or explicit "not applicable" with justification.

---

## Architecture Proposal

**Why it exists:** Translates requirements into a concrete design — components, boundaries, data flows, technology choices, trade-offs. This is where options are explored before commitment.

**Output:** Diagrams, proposal document, draft decision matrix for key choices.

**Apply:** [Thinking Framework](thinking-framework.md), [Decision Matrix](decision-matrix.md), [Organisational Constraints](organisational-constraints.md).

---

## Architecture Review

**Why it exists:** Independent scrutiny before build. The author is biased toward their own design. Review surfaces blind spots, prioritised risks, and missing trade-offs.

**Output:** Completed [Architecture Review Framework](architecture-review-framework/README.md) with P0/P1/P2 findings.

**Gate:** No unresolved P0 items without documented accepted risk and owner.

---

## ADR

**Why it exists:** Records significant decisions immutably — context, alternatives, consequences, risks. Preserves reasoning for the engineer who was not in the room.

**Output:** Accepted ADR per [template](architecture-decision-records/adr-template.md).

**When:** Technology choice, structural change, accepted risk, principle deprioritisation. See [ADR README](architecture-decision-records/README.md).

---

## Threat Model

**Why it exists:** Security review focused on assets, actors, and attack paths — not a duplicate of architecture review. Identifies controls required before exposure to production traffic or sensitive data.

**Output:** Threat model document — assets, trust boundaries, threats, mitigations, residual risk.

**Gate:** Data classification complete. Public exposure and IAM design reviewed.

**Note:** Lightweight threat modelling is acceptable for low-tier internal systems. Scope to risk.

---

## Implementation

**Why it exists:** The design becomes real — infrastructure as code, application code, configuration. Implementation must trace to approved architecture and ADRs; drift without ADR is undocumented decision-making.

**Output:** Deployed system, IaC in version control, runbooks for operational paths.

**Constraint:** No production promotion without rollback path.

---

## Validation

**Why it exists:** Proves the design meets requirements before full traffic. Includes functional testing, load testing, failover testing, security scanning, and restore testing where applicable.

**Output:** Test results, known gaps, go/no-go recommendation.

**Gate:** Tier-appropriate validation complete. Tier 1 requires load and failover evidence.

---

## Operations

**Why it exists:** The majority of a system's lifecycle is operation — deploy, monitor, incident response, patch, capacity management. Architecture that cannot be operated fails regardless of diagram quality.

**Output:** On-call runbooks, dashboards, alerts, deployment pipeline, post-incident process.

**Apply:** [Operational Excellence](engineering-principles/operational-excellence.md), [Observability](engineering-principles/observability.md).

---

## Post-Implementation Review

**Why it exists:** Compares design intent to production reality after a defined period — typically 30–90 days post-launch. Surfaces wrong assumptions, cost surprises, and operational pain before they compound.

**Output:** Review document updating ADR with production findings. Scheduled review date in ADR.

**Questions:** Did cost match estimate? Did failure modes match prediction? Can operators run it without heroics?

---

## Lessons Learned

**Why it exists:** Honest capture of what worked, what did not, and what would change. Incidents, launches, and migrations all produce lessons — this stage prevents them from staying in someone's head.

**Output:** Lessons learned section in ADR, case study, or dedicated entry. Link to incidents if applicable.

**Tone:** Blameless, specific, actionable.

---

## Architecture Revision

**Why it exists:** Systems evolve. Evidence from operations, incidents, scale, and business change may require superseding prior decisions. Revision is not failure — unrecorded revision is.

**Output:** New ADR superseding prior ADR, updated architecture, migration plan.

**Apply:** [Thinking Framework](thinking-framework.md) — "how does it evolve?" Update [Principle Conflicts](principle-conflicts.md) documentation if trade-offs shifted.

---

## Stage Skipping

| Stage | May skip when | Must document |
|-------|---------------|---------------|
| Threat Model | Internal-only, low sensitivity, no PII | Risk acceptance in ADR |
| Post-Implementation Review | Short-lived experiment, explicit sunset | Review date and sunset trigger |
| Architecture Revision | N/A — all systems eventually change | Superseding ADR when change is significant |

Skipping without documentation is not simplification — it is invisible debt.

---

## Related

- [Thinking Framework](thinking-framework.md)
- [Manifesto](manifesto.md)
- [Architecture Review Framework](architecture-review-framework/README.md)
- [ADR Template](architecture-decision-records/adr-template.md)
