# The Opare Engineering Playbook

An original engineering playbook documenting how cloud architecture is thought about, decided, operated, and improved — not how to follow a tutorial.

This repository captures **engineering judgement**: the reasoning behind platform decisions, the trade-offs accepted, the failures anticipated, and the costs understood. It is written for architects, platform engineers, and senior engineers who need to design systems that survive contact with production.

---

## Mission

To provide a structured, opinionated framework for making cloud architecture decisions — one that prioritises clarity of thought over tool familiarity, and operational reality over diagram aesthetics.

This is not a collection of copy-paste configurations. It is a record of **how to think** when building on AWS (and cloud platforms generally): what to ask, what to avoid, what to document, and how to know when a design is good enough to ship.

---

## Why This Playbook Exists

Most cloud content falls into two categories: beginner tutorials that stop at "it works on my laptop," and vendor marketing that ignores trade-offs. Neither prepares you for the questions that matter in production:

- Why this architecture and not another?
- What breaks first under load, under failure, or under budget pressure?
- What did we consciously *not* optimise for?
- How will the next engineer understand and extend this?

This playbook exists to fill that gap. It documents the **decision process** — the questions, principles, and frameworks that turn requirements into defensible architecture.

---

## What Engineering Judgement Means

Engineering judgement is the ability to make sound decisions under incomplete information, competing constraints, and real-world uncertainty. It is not intuition alone. It is a disciplined practice:

- **Context before solution.** Understand the business problem, constraints, and failure tolerance before selecting services.
- **Explicit trade-offs.** Every design choice has a cost — in complexity, latency, money, or operational burden. Name them.
- **Failure as a design input.** Systems fail. Design for graceful degradation, not perfect uptime on paper.
- **Reversibility where possible.** Prefer decisions that can be undone or evolved without rewriting the system.
- **Documentation as a deliverable.** If the reasoning isn't written down, it doesn't survive the team that made it.

Judgement improves with practice, post-incident review, and exposure to systems under load. This playbook is one tool for that practice.

---

## Repository Structure

| Directory | Purpose |
|-----------|---------|
| [`engineering-principles/`](engineering-principles/) | Foundational principles that guide every architectural decision |
| [`architecture-review-framework/`](architecture-review-framework/) | Reusable checklist for reviewing designs before build |
| [`architecture-decision-records/`](architecture-decision-records/) | ADR format and guidance for recording significant decisions |
| [`case-studies/`](case-studies/) | Placeholder for future end-to-end architecture walkthroughs |
| [`standards/`](standards/) | Documentation conventions for consistency across the playbook |
| [`references/`](references/) | External resources, standards, and further reading |

Each section is self-contained but cross-referenced. Principles inform reviews; reviews produce ADRs; ADRs and case studies reinforce principles.

---

## How Future Case Studies Will Be Written

Case studies will document **real architectural scenarios** — not toy examples. Each will follow the template in [`case-studies/case-study-template.md`](case-studies/case-study-template.md) and apply the Opare Method below.

A case study is complete when a reader can answer:

1. What problem was being solved, and for whom?
2. What constraints shaped the design?
3. Why were specific AWS services chosen over alternatives?
4. What would fail, and how would the system respond?
5. What would it cost to run, and what would it cost to change?

Case studies will include architecture diagrams, traffic flows, security and IAM design, failure analysis, cost estimates, and lessons learned. Terraform structure and testing approach will be documented where infrastructure-as-code is part of the story — but the **reasoning** remains the primary artifact.

No case studies are included in Phase 0. The structure and templates are ready for them.

---

## The Opare Method

Every significant architectural decision — and every case study — should be able to answer these eight questions:

| # | Question | What it surfaces |
|---|----------|------------------|
| 1 | **Why does this exist?** | Business purpose, stakeholder need, strategic alignment |
| 2 | **What problem are we solving?** | Scope, boundaries, success criteria |
| 3 | **What principles matter?** | Which engineering principles apply and how they constrain options |
| 4 | **What options exist?** | Viable alternatives, not just the chosen path |
| 5 | **What trade-offs are we making?** | What we gain, what we give up, what we defer |
| 6 | **What could fail?** | Failure modes, blast radius, recovery expectations |
| 7 | **What does it cost?** | Build cost, run cost, change cost, opportunity cost |
| 8 | **How does it evolve?** | Migration path, scaling path, deprecation path |

Use this method in architecture reviews, ADRs, and case studies. If you cannot answer a question, that is a signal to investigate before committing.

---

## Engineering Judgement Framework

Phase 0.5 documents — beliefs, mental models, lifecycle, and decision tools that sit above individual principles and checklists.

| Document | Purpose |
|----------|---------|
| [manifesto.md](manifesto.md) | Engineering beliefs behind the playbook |
| [thinking-framework.md](thinking-framework.md) | Mental model before making decisions |
| [engineering-workflow.md](engineering-workflow.md) | Lifecycle for every architecture in this repo |
| [decision-matrix.md](decision-matrix.md) | Reusable technology comparison template |
| [principle-conflicts.md](principle-conflicts.md) | How principles conflict and what to document |
| [organisational-constraints.md](organisational-constraints.md) | People and organisation as architectural constraints |
| [architecture-tiers.md](architecture-tiers.md) | Example Tier 1 / 2 / 3 service definitions |

Start with the [manifesto](manifesto.md) and [thinking framework](thinking-framework.md). Apply the [engineering workflow](engineering-workflow.md) when moving from problem to production.

---

## Getting Started

1. Read [`engineering-principles/README.md`](engineering-principles/README.md) for the principles that underpin all content here.
2. Use [`architecture-review-framework/README.md`](architecture-review-framework/README.md) when reviewing a new design.
3. Record decisions with [`architecture-decision-records/adr-template.md`](architecture-decision-records/adr-template.md).
4. Follow [`standards/documentation-standards.md`](standards/documentation-standards.md) when contributing.

---

## Status

**Phase 0.5 — Engineering Judgement Framework.** Principles, judgement framework, review severity model, and ADR lifecycle extensions are in place. Case studies and infrastructure examples will follow in later phases.

---

*This playbook reflects how cloud architecture is practised — with judgement, trade-offs, and accountability — not how it is taught in isolation from production reality.*
