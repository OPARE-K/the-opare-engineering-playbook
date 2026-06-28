# Engineering Manifesto

The beliefs behind this playbook. These are not slogans for a slide deck — they are constraints applied when designing, reviewing, and operating systems in production.

When a decision conflicts with a belief here, the conflict should be documented, not ignored.

---

## Engineering Judgement Over Memorising Services

Cloud services change. APIs deprecate. Certifications expire. What persists is the ability to reason under constraint: understand the problem, evaluate options, name trade-offs, and accept accountability for the outcome.

Knowing that Amazon SQS exists is not architecture. Knowing when asynchronous decoupling is worth the operational cost — and when it is not — is.

This playbook prioritises **how to think** over **what to click**.

---

## Simplicity Over Cleverness

Clever systems impress in design reviews and fail at 3am. Every abstraction, integration point, and service boundary is a future incident waiting for someone who did not attend the original meeting.

Prefer the design a competent engineer can operate six months later without calling the author. Complexity is a tax on reliability, security, and cost — paid continuously.

Simplicity is not laziness. It is the discipline of doing enough, not everything.

---

## Reliability Before Features

Features deliver value once. Reliability delivers trust continuously. A feature that ships on an unstable foundation creates rework, incident load, and customer attrition that exceeds the value of the release.

Reliability is negotiated with the business — not every system needs five nines. But the reliability target must be **explicit**, **designed for**, and **tested** before features accumulate on top.

Build the floor before the furniture.

---

## Security by Design

Security applied after launch is retrofit — expensive, incomplete, and often cosmetic. Identity, data classification, network boundaries, and auditability belong in the first sketch.

Appropriate security for the risk — not maximum security everywhere. A public marketing site and a payment API require different models. Both require conscious design.

Security failures are asymmetric. One gap can invalidate every other control.

---

## Cost as an Architectural Constraint

Cost is not a finance problem discovered after launch. Every architectural choice has a cost profile: compute, storage, transfer, licensing, operational labour, and the cost of change.

A design that is technically correct but economically unsustainable will be cut, replaced, or operated under dangerous shortcuts.

Make spend visible, attributable, and predictable. Optimise where value is created — not everywhere by default.

---

## Documentation as Part of the System

Documentation is not metadata around the real work. It is how decisions survive the team that made them, how incidents get resolved without heroics, and how the next engineer extends the system without reverse-engineering intent.

If the reasoning is not written down, it does not exist for operational purposes.

ADRs, runbooks, review records, and post-implementation reviews are **system components** — maintained, versioned, and reviewed like code.

---

## Designing for the Next Engineer

You will not be the person operating this system forever. The next engineer may have less context, less time, and no access to the original Slack thread where the decision was made.

Design and document so that someone new can understand **what** was built, **why** it was built that way, and **what not to change** without understanding the consequences.

The 3am test: if the answer to "how do we fix this?" is "call the architect," the design or its documentation needs work.

---

## Continuous Review and Improvement

Architecture is not a one-time event. Systems evolve. Traffic grows. Teams change. Incidents reveal assumptions that were wrong.

Judgement improves through feedback loops: architecture reviews before build, threat modelling before exposure, post-implementation reviews after launch, and honest retrospectives after failure.

No decision is permanent. Decisions are superseded when evidence demands it — and the supersession is recorded.

---

## Related

- [Thinking Framework](thinking-framework.md)
- [Engineering Workflow](engineering-workflow.md)
- [Principle Conflicts](principle-conflicts.md)
- [Engineering Principles](engineering-principles/README.md)
