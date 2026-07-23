---
name: arch-spec
description: Generate a technical specification for a new feature before implementation starts. Produces scope, design, API contract, and open questions so the team is aligned before a single line of code is written. Triggers on: "write spec", "technical spec", "feature spec", "spec for".
---

# arch-spec: Technical Feature Specification

Write a technical specification for: `$ARGUMENTS`

**Step 1** — Read relevant existing code, types, routes, and services to understand the current architecture before designing anything new.

## Spec Structure

```markdown
# Feature: [Name]

## Problem Statement
What user problem does this solve? What breaks or is missing without it?

## Scope

### In scope
- [Concrete thing 1]
- [Concrete thing 2]

### Out of scope (explicitly)
- [Thing that might be assumed but is NOT included]

## Design

### Data model
What new or modified types/interfaces are needed?

\`\`\`typescript
interface ExampleEntity {
  id: string
  // ...
}
\`\`\`

### Azure resources
Any new or modified resources? (Functions, Table Storage tables, SWA config, CORS)

### API contract
New or modified endpoints — use arch-api-design format for each.

### Frontend changes
Components added/modified, state changes, routing.

## Implementation Plan

Ordered task list — each task independently committable:
1. [Task 1]
2. [Task 2]
...

## Testing Plan
- Unit tests: what to cover
- E2E/smoke: what user flows to verify
- Manual verification: golden path steps

## Open Questions
- [ ] [Decision needed before implementation can start]
- [ ] [Assumption that needs validation]

## Not Decided Yet
Things that will be resolved during implementation — document them so they don't get forgotten.
```

## Quality Bar

- Scope section explicitly lists what is NOT included — prevents scope creep
- Data model and API contract are concrete enough that implementation has no ambiguity
- Implementation plan is ordered by dependency — no task assumes unfinished prior work
- Open questions are flagged before implementation starts, not discovered mid-sprint

---

**Output the completed spec. Implementation does not start until open questions are resolved.**
