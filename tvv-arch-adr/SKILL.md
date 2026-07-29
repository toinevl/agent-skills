---
name: tvv-arch-adr
description: 'Draft an Architecture Decision Record (ADR) for a technical decision with options analysis, trade-offs, and a clear recommendation. Use when making a significant technical choice that future contributors should understand. Triggers on: "write ADR", "document decision", "architecture decision", "ADR".'
version: 1.0.0
category: architecture
scope: [all]
triggers: [write ADR, document decision, architecture decision, ADR]
status: stable
created: '2026-07-23'
last_updated: '2026-07-29'
---

# tvv-arch-adr: Architecture Decision Record

Draft an ADR for the decision described in `$ARGUMENTS`.

**Step 1** — Read relevant existing code and any prior ADRs in `docs/adr/` (or equivalent) to understand context and numbering.

## Format

```markdown
# ADR-[number]: [Short title — what was decided]

Date: YYYY-MM-DD
Status: Proposed | Accepted | Deprecated | Superseded by ADR-[n]

## Context

What problem are we solving? What forces are at play?
(technical constraints, team size, budget, existing architecture, timeline)

## Options Considered

### Option A: [Name]
[Brief description]
- Pros: ...
- Cons: ...

### Option B: [Name]
[Brief description]
- Pros: ...
- Cons: ...

### Option C: [Name] (if applicable)
...

## Decision

We chose **Option [X]** because [one clear sentence on the deciding factor].

## Consequences

**Positive:**
- ...

**Negative / trade-offs accepted:**
- ...

**Risks:**
- ...

## References

- [Link to relevant docs, issues, or prior art]
```

## Tips for Good ADRs

- Title states the decision, not the problem ("Use Azure Table Storage for telemetry" not "Storage decision")
- Context section is honest about constraints — budget, time, team skill all count
- List at least 2 real options; if there was truly only one, say why alternatives were ruled out immediately
- Consequences section is candid about downsides — future readers need to know what was accepted
- Keep it short enough to read in 3 minutes

---

**Output the completed ADR ready to save as `docs/adr/ADR-[number]-[slug].md`.**
