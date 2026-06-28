# Organisational Constraints

Architecture is not designed in a vacuum. The best technical solution that ignores who builds it, who runs it, and what the organisation can sustain will fail — not in theory, in operations.

Evaluate every design against these constraints alongside technical requirements. See [Thinking Framework](thinking-framework.md) — "who operates it?" and [Decision Matrix](decision-matrix.md) — context section.

---

## Team Size

Small teams cannot operate large distributed systems safely. Every service boundary, deployment pipeline, and on-call rotation consumes people.

- How many engineers will build and maintain this?
- Is there dedicated platform or SRE support, or is the product team full-stack responsible?
- What happens when one person is on leave or leaves?

**Design implication:** Prefer managed services, fewer moving parts, and consolidated ownership when team size is small. Microservices multiply operational surfaces linearly with service count.

---

## Team Skill Level

The correct architecture for a team expert in Kubernetes differs from a team strong in serverless and managed databases.

- What does the team operate confidently today?
- What would require hiring, training, or consultants?
- Is the skill gap closable within the project timeline?

**Design implication:** Match technology to existing capability plus realistic upskilling — not aspirational org charts.

---

## Ownership

Unclear ownership produces unmaintained systems, alert fatigue with no responder, and decisions no one can approve.

- Who owns runtime health — on-call, patching, incidents?
- Who owns the roadmap — features vs reliability vs cost?
- Who approves architectural change?

**Design implication:** Every production system has a named owner before launch. Shared ownership without a primary is no ownership.

---

## On-Call Capacity

On-call is a finite resource. Alert design, system complexity, and runbook quality directly affect burnout and MTTR.

- Who is paged and how often is acceptable?
- Is there escalation beyond the building team?
- Can the team support 24/7 for this tier, or is business-hours support explicit?

**Design implication:** Tier 1 expectations require Tier 1 on-call capacity. Do not assign 99.9% availability to a system no one pages for.

See [Architecture Tiers](architecture-tiers.md).

---

## Budget

Budget constrains build time, run cost, and tool choices.

- Cap on monthly infrastructure spend?
- Headcount for build vs buy decisions?
- Approval threshold for recurring cost increase?

**Design implication:** Document cost at launch and projected scale. A design that exceeds budget will be cut or operated unsafely.

See [Cost Awareness](engineering-principles/cost-awareness.md).

---

## Compliance

Regulatory and contractual obligations may mandate controls that shape architecture — data residency, retention, audit, encryption, access review.

- Which frameworks apply — SOC 2, PCI, HIPAA, GDPR, industry-specific?
- Who attests compliance — security, legal, external auditor?
- What evidence is required — logs, configs, process documentation?

**Design implication:** Compliance requirements are constraints, not optional security extras. Identify early in [Engineering Workflow](engineering-workflow.md).

---

## Approval Paths

Organisations gate change through architecture review boards, security review, finance sign-off, or change advisory boards.

- What reviews are mandatory before production?
- Who can approve accepted risk?
- How long do approvals take — does timeline account for this?

**Design implication:** Factor approval latency into delivery plans. Designs that bypass process create audit and operational debt.

---

## Deployment Frequency

Desired release cadence shapes architecture — monolith vs microservices, feature flags, database migration strategy, blue/green capability.

- How often must the business ship?
- Can the architecture support that cadence safely?
- What is the current change failure rate tolerance?

**Design implication:** Weekly deploys require different tooling and test strategy than quarterly big-bang releases.

---

## Support Model

Define who users and internal teams contact when something breaks — self-service, ticket queue, dedicated support, account team.

- Internal platform vs external customer support?
- SLA for response time by tier?
- Handoff between L1 support and engineering?

**Design implication:** Runbooks and observability must match support model — L1 cannot diagnose without symptom-based alerts and clear escalation.

---

## Operational Maturity

Honest assessment of where the organisation sits — ad hoc operations, repeatable processes, defined metrics, continuous improvement.

| Maturity signal | Lower maturity | Higher maturity |
|-----------------|----------------|-----------------|
| Deployments | Manual, heroic | Automated, rollback tested |
| Incidents | Repeated surprises | Blameless postmortems, action items closed |
| Monitoring | Reactive user reports | SLOs, error budgets, paging on symptoms |
| Documentation | Tribal knowledge | ADRs, runbooks, current diagrams |
| DR | Untested backups | Scheduled failover drills |

**Design implication:** Do not design for Google SRE maturity on a team still establishing CI/CD. Meet the organisation where it is — with a documented path to improve.

---

## Using Constraints in Review

At architecture review, explicitly capture:

| Constraint | Current state | Impact on design |
|------------|---------------|------------------|
| Team size | | |
| On-call | | |
| Budget | | |
| Compliance | | |
| Operational maturity | | |

A design that violates organisational constraint without mitigation plan should be **P1 Significant Risk** or **P0 Launch Blocker** depending on severity.

See [Architecture Review Framework](architecture-review-framework/README.md).

---

## Related

- [Thinking Framework](thinking-framework.md)
- [Engineering Workflow](engineering-workflow.md)
- [Principle Conflicts](principle-conflicts.md)
- [Architecture Tiers](architecture-tiers.md)
