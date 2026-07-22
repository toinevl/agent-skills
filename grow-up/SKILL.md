---
name: grow-up
description: Turn problems into permanent process improvements. Use this skill whenever something breaks, a mistake happens, a discipline is violated, or a pattern fails—in any project, for any type of problem. The skill performs root cause analysis, identifies why it happened, and automatically updates skills, CLAUDE.md rules, and config to prevent recurrence. Automates the full learning cycle so the same mistake never happens the same way twice. Invoke this skill whenever you notice a problem that should have been caught, whenever a user or team member asks "why did this happen", whenever you're about to file a bug that could be prevented by better process.
---

# Grow-Up: Learn From Problems & Prevent Recurrence

## Philosophy

Most problems aren't one-time accidents—they're symptoms of broken systems. The "grow-up" skill turns every problem into a permanent improvement: analyze what went wrong, fix the root cause, and update the tools/processes/skills so it can't happen the same way again.

**The cycle:**
1. Problem occurs
2. Root cause analysis (flexible, not rigid)
3. Identify the fix type (skill, rule, config, automation)
4. Automate the fix
5. Commit & document
6. System is now stronger

---

## Step 1: Describe the Problem

When you invoke this skill, clearly describe:

- **What happened?** The symptom/event (e.g., "Created docs without adding to wishlist first")
- **When?** Time/context (e.g., "During deployment phase")
- **Who noticed?** (e.g., "User pointed out during review")
- **Impact?** (e.g., "Backlog diverged from actual work, teammates couldn't see progress")

Example:
```
Problem: I created DEPLOY_CHECKLIST.md and MONITORING_SETUP.md without adding 
them to wishlist.md first, breaking the "wishlist is single source of truth" 
principle.

When: During Strategy 1 deployment phase work

Who: User explicitly called it out

Impact: Wishlist diverged from actual work, lane coordination was unclear, 
user had to ask why docs weren't tracked
```

---

## Step 2: Root Cause Analysis (Flexible)

The skill will guide you through root cause analysis. This is **not** a rigid formula—adapt as needed.

### Common frameworks (pick what fits):

**5 Whys:**
```
Q: Why was the problem not caught?
A: I didn't check the wishlist before creating docs

Q: Why didn't you check?
A: I treated docs as deliverables (outputs), not work items (backlog)

Q: Why that mental model?
A: No pre-work discipline or checklist

Q: Why no checklist?
A: Nothing enforced it at the start

Q: Why wasn't it enforced?
A: Wishlist was secondary reference, not primary coordination tool
```

**Fishbone Diagram (causes):**
```
Problem: Wishlist diverged from work

← People: No pre-work discipline, mental model mismatch
← Process: No checklist before implementation, wishlist not checked first
← Tools: Wishlist not integrated into workflow
← Environment: No enforcement mechanism, no skill to guide behavior
```

**Layer 3 Analysis:**
```
Layer 1 (Symptom): Docs created without wishlist entry
Layer 2 (Cause): Didn't check/add to wishlist before writing
Layer 3 (Root): Mental model treated docs as outputs, not work items; 
                no enforcement or pre-flight check
```

### What the skill will do:

1. Ask you questions about the problem
2. Help identify patterns (is this a one-off or systemic?)
3. Surface the root cause (usually: missing guard, bad mental model, no automation, unclear trigger)
4. Summarize in a clear statement

---

## Step 3: Identify the Fix Type

Based on the root cause, the skill determines what needs to be fixed:

| Root Cause | Fix Type | Example |
|---|---|---|
| **Missing guard/check** | Automation + skill | Add pre-flight checklist skill |
| **Bad mental model** | CLAUDE.md rule + skill | Add "wishlist-first" discipline to CLAUDE.md |
| **No trigger/enforcement** | Skill description | Create skill with clear trigger conditions |
| **Process missing** | CLAUDE.md + workflow | Add new section to CLAUDE.md |
| **Config/setup issue** | Config update | Update .env, bicep, github workflows |
| **Repeated pattern** | Bundled script | Extract repeated work into reusable script |
| **Multiple causes** | Skill + rule + config | Combine fixes |

The skill will ask: "What type of fix best prevents this?" and help you choose.

---

## Step 4: Automate the Fix

### If the fix is a **new skill:**

The skill will:
1. Create skill directory: `~/.claude/skills/<skill-name>/`
2. Draft SKILL.md with:
   - Name, description (pushy trigger)
   - Instructions for the problem area
   - Examples and patterns
   - Guards/assertions to prevent recurrence
3. Create `evals/evals.json` with test cases
4. Save to filesystem
5. Verify it's registered

### If the fix is a **CLAUDE.md rule:**

The skill will:
1. Identify which project's CLAUDE.md needs updating (or create one)
2. Add new section with:
   - Rule/principle in clear language
   - Why it matters (theory of mind)
   - How to apply it
   - Examples (good vs bad)
3. Update the file
4. Verify changes
5. Commit with message explaining the rule

### If the fix is a **config/automation:**

The skill will:
1. Identify the config file (.github/workflows, bicep, .env, etc.)
2. Add guards, checks, or automation
3. Test the change works
4. Commit

### If the fix is **multiple** (skill + rule + config):

The skill chains them together.

---

## Step 5: Verify & Commit

The skill will:

1. **Verify changes:**
   - New skill is syntactically correct
   - CLAUDE.md is valid markdown
   - Config is valid YAML/JSON
   - Files are in the right location

2. **Commit with good message:**
   ```
   grow-up: learn from [problem] + prevent recurrence
   
   Root cause: [3-sentence summary]
   
   Fix: [What was changed/created]
   - Created skill: ...
   - Updated CLAUDE.md: ...
   - Config change: ...
   
   This prevents: [How it stops the problem]
   
   Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
   ```

3. **Document the fix:**
   - Skill includes examples of the problem (bad pattern)
   - CLAUDE.md rule explains the why
   - Commit message is searchable for future reference

---

## Example: The Wishlist Problem

**Input:**
```
Problem: Created DEPLOY_CHECKLIST.md and MONITORING_SETUP.md without adding 
to wishlist.md first

Root cause: Mental model treated docs as deliverables, not work items; 
no pre-flight discipline or enforcement
```

**What the skill did:**

1. **Created skill: `roomsense-wishlist-first`**
   - Pre-work checklist (add to wishlist before code)
   - Commit discipline (reference wishlist in message)
   - Post-work verification (mark complete, add SHA)
   - Lane coordination (show @C/@H/@O ownership)
   - Example patterns

2. **Updated CLAUDE.md**
   - New section: "Wishlist-First Discipline"
   - Explains why (visibility, lane coordination, no invisible work)
   - Process: Before → During → Commit → After
   - Guard: "If creating file without wishlist entry, fix immediately"

3. **Committed:**
   ```
   grow-up: learn from wishlist divergence + prevent recurrence
   
   Root cause: Mental model treated docs as outputs, not work items.
   No pre-flight discipline or enforcement.
   
   Fix:
   - Created skill: roomsense-wishlist-first (checklist + guidance)
   - Updated CLAUDE.md: Added "Wishlist-First Discipline" section
   - Formalized rule: Every work item on wishlist BEFORE implementation
   
   This prevents: Wishlist divergence, invisible work, broken lane coordination
   
   Co-Authored-By: Claude Haiku 4.5 <noreply@anthropic.com>
   ```

**Result:** Same mistake can't happen the same way twice. The system learned.

---

## How It Works (Full Workflow)

### 1. You notice a problem

```
User says: "Why is this on the wishlist? I thought it was already done?"
or
You realize: "Wait, we just violated the no-false-affordance rule"
or
System shows: "Tests pass but behavior is broken"
```

### 2. Invoke the skill

```
"I just made a mistake: [describe]. Can you do a root cause analysis
and update the tools/rules to prevent this?"
```

### 3. Skill asks clarifying questions

- What caused it? (Mental model? Missing check? No automation?)
- Is this one-off or systemic?
- What's the best fix? (Skill? Rule? Config? Guard?)
- Which project? (RoomSense? Other?)

### 4. Skill automates the fix

- Creates/updates skill
- Updates CLAUDE.md or config
- Tests changes
- Commits

### 5. System is stronger

- Next time, the mistake is caught earlier or prevented entirely
- Team members see the new rule/skill and understand why
- Future work is guided by the learned lesson

---

## What Gets Created/Updated

### New Skills

Stored in: `~/.claude/skills/<skill-name>/`

Includes:
- `SKILL.md` — Instructions, examples, guards
- `evals/evals.json` — Test cases to validate it works
- Optional: `references/` docs, `scripts/` helpers

### CLAUDE.md Updates

Updated: `<project>/CLAUDE.md` or `<repo>/.claude/CLAUDE.md`

Includes:
- New section with rule/principle
- Why it matters
- How to apply
- Good vs bad examples
- Related skills/guards

### Config Updates

Examples:
- `.github/workflows/*.yml` — Add automation/checks
- `bicep/` files — Add guards
- `.env.example` — Document new requirements
- Scripts — Add validation

---

## Edge Cases & Safety

### What the skill WON'T do:

- ❌ Overwrite existing files without confirmation
- ❌ Create breaking changes to critical systems
- ❌ Delete code (only add guards/prevention)
- ❌ Commit without clear message

### What it WILL do:

- ✅ Add guards and checks
- ✅ Create new skills/rules
- ✅ Update config safely (append, don't replace)
- ✅ Ask before making major changes
- ✅ Test changes before committing

---

## Repeated Invocation

If the same problem happens **twice**, the skill notices and:

1. Updates the skill/rule to be more aggressive
2. Adds additional checks
3. Escalates the priority
4. Documents the pattern

Example:
```
"Created docs without wishlist entry AGAIN (third time)"

Skill response:
- Updates skill description to be more pushy
- Adds automated pre-commit check in .git/hooks/
- Adds CI/CD lint rule to catch wishlist violations
- Escalates to team-wide enforcement

Message: "This pattern keeps happening. I've made prevention automatic."
```

---

## When to Use This Skill

### ✅ Perfect for:
- You realize a mistake you made
- User points out something broke
- A discipline rule was violated
- A pattern failed
- An assumption was wrong
- A guard failed to catch something

### ⏸️ Not needed for:
- One-time typo (grep fix it, move on)
- Temporary debug code (just remove it)
- Clear user error (guide them, don't create a skill)

### 🤔 Edge case:
- Issue happens in production vs development
- Critical vs non-critical
- One project vs all projects

The skill helps you decide.

---

## Quick Start

**When you notice a problem:**

```
"I made a mistake: [describe problem]

Root cause: [what made this possible]

Can you do a root cause analysis and create/update the tools to prevent it?"
```

**The skill will:**

1. Ask clarifying questions (if needed)
2. Do the analysis
3. Create/update skills or rules
4. Commit the changes
5. System is now stronger

**Example response:**
```
✅ Root cause identified: Mental model mismatch (docs as outputs, not items)

✅ Fix created: New skill "roomsense-wishlist-first" with pre-work checklist

✅ Rule updated: Added "Wishlist-First Discipline" to CLAUDE.md

✅ Committed: grow-up commit explaining the learning

🎯 Result: Same mistake cannot happen the same way twice
```

---

## Philosophy Summary

**The grow-up principle:** Every mistake is a teacher. Extract the lesson, automate the prevention, commit the knowledge, and move forward stronger.

Not: "Oops, I'll be more careful next time."  
But: "Oops, the system is now designed so this can't happen this way again."

The goal is a self-healing, self-improving codebase where mistakes become permanent improvements.
