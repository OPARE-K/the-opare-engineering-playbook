# Documentation Standards

Conventions for writing and maintaining content in this playbook. Consistency makes the repository navigable for readers and sustainable for contributors — including your future self.

These standards apply to all markdown in this repository unless a specific template overrides them.

---

## Voice and Tone

Write as a **senior engineer explaining to peers** — precise, direct, and opinionated where judgement matters. Avoid:

- Tutorial language ("In this section we will learn...")
- Marketing superlatives ("best-in-class," "world-class")
- Unexplained acronyms on first use
- Passive voice where active is clearer

Prefer:

- Declarative statements of decision and rationale
- Explicit trade-offs and accepted risks
- Concrete examples where they clarify abstract principles
- Short paragraphs and scannable structure

---

## Document Structure

### Headings

- One `#` title per document — the document name
- Use `##` and `###` for hierarchy; do not skip levels
- Keep heading titles descriptive ("Failure Analysis" not "Section 4")

### Opening Paragraph

Every standalone document should open with **what this document is and who it is for** in one or two sentences — except templates, which use placeholder instructions.

### Sections

Long documents should include:

1. Context or purpose
2. Main content (principles, checklist, narrative)
3. Related links to other playbook sections

---

## Formatting Conventions

| Element | Convention |
|---------|------------|
| **File names** | Lowercase, kebab-case: `security-by-design.md` |
| **ADR files** | Numbered prefix: `0001-short-title.md` |
| **Case study dirs** | Numbered prefix: `01-scenario-name/` |
| **Links** | Relative paths within repo: `[Reliability](../engineering-principles/reliability.md)` |
| **Tables** | Use for comparisons, checklists with columns, structured metadata |
| **Code blocks** | For directory trees, CLI examples, review output templates — not for prose |
| **Diagrams** | ASCII or linked images; include alt text or description for accessibility |

---

## Opare Method Integration

Documents that describe decisions (ADRs, case studies, review outputs) should map content to the [Opare Method](../README.md#the-opare-method) either explicitly or by section alignment:

| Method question | Typical location |
|-----------------|------------------|
| Why does this exist? | Business problem, context |
| What problem are we solving? | Requirements, scope |
| What principles matter? | Principles cited in ADR/case study |
| What options exist? | Alternatives considered |
| What trade-offs? | Trade-offs section, consequences |
| What could fail? | Failure analysis, risks |
| What does it cost? | Cost implications / cost analysis |
| How does it evolve? | Future improvements |

If a section is intentionally empty (e.g., no Terraform yet), state that and why.

---

## ADR Conventions

- Use the [ADR template](../architecture-decision-records/adr-template.md) without removing required sections — mark N/A if truly not applicable
- Status must reflect reality: Proposed until formally accepted
- Supersede; do not edit accepted ADRs in place for material changes
- Link related ADRs, case studies, and review notes

---

## Case Study Conventions

- Follow the [case study template](../case-studies/case-study-template.md)
- One primary `README.md` per case study directory
- Link to ADRs for decisions; do not duplicate full ADR text in the case study
- Include at least one architecture diagram and one traffic flow diagram when published
- Cost figures should be labelled as estimates with assumptions stated

---

## Checklist and Review Documents

- Use `- [ ]` for unchecked items in markdown checklists
- When archiving a completed review, change relevant items to `- [x]` or summarise outcome in a review summary block
- Record blockers, accepted risks, and action items explicitly

---

## What Not to Document Here

- Step-by-step AWS console tutorials
- Credentials, account IDs, or internal-only URLs
- Copy-paste Terraform or application code without architectural narrative (case studies may reference structure, not dump modules)
- Content that duplicates AWS documentation — link instead

---

## Maintenance

- Review principles and framework annually or after major cloud platform shifts
- Update "Revisit when" triggers in ADRs when conditions change
- Archive or mark deprecated content rather than deleting without trace — link to successor

---

## Related Resources

- [Root README](../README.md)
- [ADR README](../architecture-decision-records/README.md)
- [Case Studies README](../case-studies/README.md)
