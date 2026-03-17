---
name: itkdev-adr
description: Use when creating, updating, or managing Architecture Decision Records (ADRs), documenting architectural decisions, or when discussion reveals a significant technical decision that should be recorded.
---

# Architecture Decision Records (ADR)

You are helping create and manage Architecture Decision Records. ADRs capture important architectural decisions along with their context and consequences.

For more information about ADRs, see [adr.github.io](https://adr.github.io/).

## Information Gathering

**Before writing an ADR, ask questions to gather the necessary information.** Use the AskUserQuestion tool to clarify:

1. **What decision needs to be documented?**
   - What is the problem or challenge being addressed?
   - What triggered this decision?

2. **Who is involved?**
   - Who is making the decision?
   - Who are the stakeholders (advisors and affected parties)?

3. **What options were considered?**
   - What alternatives were evaluated?
   - What are the pros and cons of each option?

4. **What was decided and why?**
   - Which option was chosen?
   - What is the rationale for this choice?

5. **What are the consequences?**
   - What are the benefits of this decision?
   - What are the trade-offs or risks?
   - Are there follow-up actions needed?

Ask these questions conversationally before drafting the ADR. This ensures all required fields can be properly filled in.

## When to Create an ADR

Create an ADR when making decisions about:
- Technology or framework selection
- Architectural pattern choices
- Integration approach decisions
- Security-related decisions
- Breaking changes or deprecations
- Cross-team impacting decisions
- Any decision that would be useful to understand later

## ADR Template

Use this format for all ADRs:

```markdown
# NNN: [Decision Title]

| Field | Value |
|-------|-------|
| **Created By** | [Author Name] |
| **Date** | YYYY-MM-DD |
| **Decision Maker** | [Name(s)] |
| **Stakeholders** | [Advisors and affected parties] |
| **Status** | Draft |

## Context

[Problem statement and background - what situation or challenge prompted this decision?]

### Drivers

- **Functional:** [Requirements that influence the decision]
- **Non-functional:** [Quality attributes: performance, security, maintainability, etc.]

### Options Considered

1. **Option A**: [Description with pros/cons]
2. **Option B**: [Description with pros/cons]
3. **Option C**: [Description with pros/cons]

## Decision

[Chosen option and detailed rationale for why this option was selected over alternatives]

## Consequences

### Positive
- [Benefit 1]
- [Benefit 2]

### Negative / Trade-offs
- [Trade-off 1]
- [Trade-off 2]

### Follow-up Actions
- [ ] [Action item if any]
```

## File Organization

- **Location:** `docs/adr/` (create directory if needed)
- **Naming:** `NNN-brief-title.md` (e.g., `001-database-selection.md`)
- **Numbering:** Three-digit sequential numbers starting at 001
- **Index:** Maintain a `docs/adr/README.md` listing all ADRs

### Index Template (docs/adr/README.md)

```markdown
# Architecture Decision Records

This directory contains Architecture Decision Records (ADRs) for this project.

| Number | Title | Status | Date |
|--------|-------|--------|------|
| [001](001-example-decision.md) | Example Decision | Accepted | YYYY-MM-DD |
```

## Status Values

- **Draft** - Decision is being discussed, not yet finalized
- **Accepted** - Decision has been approved and should be followed
- **Rejected** - Decision was considered but not adopted
- **Deprecated by NNN** - Decision has been replaced (link to new ADR)
- **Supersedes NNN** - This decision replaces a previous one (link to old ADR)

## Workflow

### Creating a New ADR

1. Determine the next ADR number by checking existing ADRs in `docs/adr/`
2. Create the file with format `NNN-brief-title.md`
3. Fill in all template sections
4. Set status to "Draft"
5. Update the index in `docs/adr/README.md`
6. Create a PR for review

### Accepting an ADR

1. Update status from "Draft" to "Accepted"
2. Ensure all sections are complete
3. Verify stakeholders have reviewed

### Deprecating an ADR

1. Update the old ADR's status to "Deprecated by NNN"
2. Create a new ADR that "Supersedes NNN"
3. Update both entries in the index

## Integration with GitHub Workflow

When creating ADRs:
- Follow `itkdev-github-guidelines` for commits and PRs
- Reference related GitHub issues in the ADR context
- Use commit message: `docs: add ADR NNN - brief title`
- PR should explain why this decision is being documented
