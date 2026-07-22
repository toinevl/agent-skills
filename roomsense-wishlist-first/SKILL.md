---
name: roomsense-wishlist-first
description: Enforce wishlist-first discipline for RoomSense backlog management. Use this skill BEFORE starting any work item (code, docs, tests, infra) to ensure it's tracked on wishlist.md. The wishlist is the single source of truth for progress across all lanes (@C frontend, @H backend, @O orchestrator). Never create work without a corresponding wishlist entry. Include this skill whenever the user is about to start a new RoomSense task, feature, or doc, or when reviewing commits to verify wishlist discipline.
---

# RoomSense Wishlist-First Discipline

## Why This Matters

`wishlist.md` is the single source of truth for RoomSense progress. Without it:
- ❌ Progress becomes invisible (teammates can't see what's done)
- ❌ Lane coordination fails (Hermes doesn't know Claude's status)
- ❌ Duplicates and conflicts emerge
- ❌ Status queries require reading git history instead of the backlog

**The Rule:** Every work item (code, docs, tests, infrastructure) MUST be on `wishlist.md` BEFORE work starts.

---

## Pre-Work Checklist

When starting a new task, complete this checklist BEFORE implementation:

### 1. Add to Wishlist (Before Writing Code)

- [ ] Open `wishlist.md`
- [ ] Find or create the relevant section (Strategy #X, Phase, or new item)
- [ ] Add line with `[ ]` (unchecked) + description + lane owner (@C, @H, @O)
  ```
  - [ ] (Priority) your-task-name +category @lane #number — in progress YYYY-MM-DD
    - [ ] Sub-task 1 (@lane)
    - [ ] Sub-task 2 (@lane)
  ```
- [ ] Reference the item in your work (e.g., "working on #36")

**Example:**
```
- [ ] (C) confirmation modal component (48px buttons, slide animation) +mobile @C #36a
  - [ ] Create component (confirmationModal.ts)
  - [ ] Write tests (6 tests: render, buttons, callbacks, overlay)
  - [ ] Integrate into roomFinder.ts
  - [ ] Verify all tests pass
```

### 2. Commit with Wishlist Reference

When committing:
- [ ] Include wishlist item number in commit message: `feat(#36a): add modal`
- [ ] Reference specific lines that were added/changed

**Example:**
```bash
git commit -m "feat(#36a): add booking confirmation modal

Components:
- confirmationModal: overlay modal with room details + buttons
- Tests: 6 passing (render, buttons, callbacks, overlay, occupancy)

Wishlist #36a sub-tasks:
- [x] Create component (confirmationModal.ts)
- [x] Write tests (confirmationModal.test.ts)
- [x] Integrate into roomFinder.ts
- [ ] Browser test on 3G throttle (next)"
```

### 3. Mark Complete When Done

After finishing:
- [ ] Open `wishlist.md`
- [ ] Find your item and mark `[x]`
- [ ] Add commit SHA(s) on the same line
  ```
  - [x] (C) confirmation modal component (2f41ceb; 6 tests passing) (@C)
  ```

---

## Post-Commit Verification

After pushing, verify wishlist discipline:

### Check 1: Wishlist Entry Exists
```bash
grep "#36" wishlist.md  # Should find your item
```

### Check 2: Commit References Wishlist
```bash
git log --oneline -5 | grep "#36"  # Should see reference in commit
```

### Check 3: Item Marked Complete
```bash
grep "\[x\].*#36" wishlist.md  # Should find [x] if done
```

---

## Lane Coordination

Use lane owners in wishlist to coordinate:

| Lane | Owner | Owns | Example |
|------|-------|------|---------|
| **@C** | Claude (frontend) | `frontend/**`, pages, components, styling | `- [ ] (C) mobile modal (@C #36)` |
| **@H** | Hermes (backend) | `api/**`, functions, database | `- [ ] (B) presence endpoint (@H #37a)` |
| **@O** | Orchestrator | `infra/**`, `.github/workflows/**`, `packages/shared` | `- [ ] (A) deploy workflow (@O #22)` |

**Example of parallel work:**
```
- [ ] (C) strategy 2: social presence UX +extend @C @H #37
  - **Phase 2a: Backend (Hermes blocks avatar stack)**
    - [ ] UserPresence table (@H)
    - [ ] GET /api/presence endpoint (@H)
  - **Phase 2b: Frontend (Claude, after 2a)**
    - [ ] PresenceIndicator component (@C, mock data first)
    - [ ] Consent modal (@C)
```

Hermes knows to start on "Phase 2a", Claude waits for the API before building "Phase 2b". Wishlist makes this explicit.

---

## What NOT To Do

### ❌ Bad: Silent Work

```bash
# Write code without adding to wishlist
echo "great new feature!" > frontend/src/newFeature.ts
git add frontend/src/newFeature.ts
git commit -m "feat: new feature"  # No wishlist reference!
```

**Problem:** Progress is invisible. Teammates don't know this is done.

### ❌ Bad: Wishlist Update AFTER Commit

```bash
git commit -m "add modal"  # No wishlist reference
# Then remember: "oh I should update wishlist"
# Edit wishlist afterward
```

**Problem:** Wishlist lags behind work. Not the source of truth.

### ❌ Bad: Creating Docs Without Tracking

```bash
# Created DEPLOY_CHECKLIST.md and MONITORING_SETUP.md
# Without adding to wishlist first
# User has to ask: "why isn't this on the wishlist?"
```

**Problem:** Discipline breaks down. User can't see what you're building.

### ✅ Good: Wishlist First

```bash
# 1. Add to wishlist
vi wishlist.md  # Add "- [ ] (C) deployment docs (@C #36c)"

# 2. Create docs
cat > DEPLOY_CHECKLIST.md << EOF
...
EOF

# 3. Commit with reference
git add DEPLOY_CHECKLIST.md wishlist.md
git commit -m "docs(#36c): add deployment checklist

Checklist includes:
- Pre-deployment verification
- Post-deployment smoke tests
- Monitoring setup
- Rollback procedures"

# 4. Mark complete
vi wishlist.md  # Change to "- [x] (C) deployment docs (f67adf3) (@C)"
git add wishlist.md
git commit -m "docs: mark #36c complete"
```

---

## Pattern: Priority & Status

Wishlist entries follow this pattern:

```
- [ ] (PRIORITY) description +category @lane #number — status YYYY-MM-DD
```

### Priority (A-D)
- **(A)** Blocking (other items depend on this)
- **(B)** High (should do soon)
- **(C)** Medium (should do)
- **(D)** Low (nice to have)

### Status
- `in progress YYYY-MM-DD` — being worked on
- `done YYYY-MM-DD` — completed with `[x]`
- `blocked` — waiting for something
- `paused` — intentionally stopped

### Category Tags (+tag)
- `+mobile` — mobile UX
- `+extend` — extending existing feature
- `+bug` — bug fix
- `+docs` — documentation
- `+infra` — infrastructure

### Example:
```
- [x] (B) confirmation modal component (2f41ceb) +mobile @C #36a — done 2026-07-22
  - [x] Create component (confirmationModal.ts)
  - [x] Write tests (confirmationModal.test.ts)
  - [x] Integrate into roomFinder.ts
```

---

## Quick Reference: Pre-Work → Post-Work

**Before you start:**
```
1. vi wishlist.md → Add item with [ ]
2. Start implementation
```

**When you commit:**
```
1. git commit -m "feat(#36a): description"
2. Reference #36a in commit message
```

**When you're done:**
```
1. vi wishlist.md → Change [ ] to [x]
2. Add commit SHA
3. git add wishlist.md && git commit -m "docs: mark #36a complete"
```

---

## Common Questions

**Q: What if the task is tiny (5 lines of code)?**  
A: Still add to wishlist. Even small changes should be tracked. It's discipline, and scale doesn't change that.

**Q: Can I add sub-tasks later?**  
A: Yes. Add the main item first, then add sub-tasks as they emerge. But the main item must exist BEFORE work starts.

**Q: What if I'm fixing a typo in a doc?**  
A: If it's truly trivial (one word, no behavior change), a commit without a wishlist item is OK. But if it's any real content or code, add it.

**Q: What if the work was requested verbally and not on wishlist yet?**  
A: Add it to wishlist FIRST, then do the work. The wishlist entry is your "contract" for what you're building.

**Q: How do I know when to mark something [x]?**  
A: When you've done the work AND the commit is pushed to main. Not before.

---

## Enforcing This in Code Review

When reviewing commits:

```bash
# Check if commit references wishlist
git log --oneline <branch> | grep "#"

# Check if wishlist was updated
git log --oneline <branch> | grep "docs: mark.*complete"

# Check if items are marked [x]
git diff main wishlist.md | grep "\[x\]"
```

If a commit has no wishlist reference:
```
❌ Commit: "feat: add new button"
✅ Should be: "feat(#36): add new button"

Please also update wishlist.md to mark #36 [x] and add commit SHA
```

---

## Summary

| Step | Action | Example |
|------|--------|---------|
| **Before** | Add to wishlist | `- [ ] (C) modal component @C #36` |
| **During** | Code & test | Create files, write tests |
| **Commit** | Reference item | `git commit -m "feat(#36): add modal"` |
| **After** | Mark complete | `- [x] (C) modal (2f41ceb) @C #36` |

This discipline ensures:
- ✅ Progress is always visible (check wishlist, not git)
- ✅ Teammates know who's doing what (@C, @H, @O)
- ✅ No invisible work
- ✅ Wishlist is the source of truth

**The mantra:** *Wishlist first, code second. Always.*
