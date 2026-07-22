---
name: sync-skills
description: Keep your local skills repo updated from GitHub. Use this skill before invoking other skills to ensure you have the latest versions. Works for Claude and Hermes—automates cloning/pulling from the skills repository, updates local copy, shows what changed, and verifies all skills are ready. Make sure to use this skill whenever you're about to use other skills, or whenever you want to ensure you have the latest skill definitions and improvements.
---

# sync-skills: Keep Skills Repo Updated

## Purpose

The skills repo on GitHub is the source of truth for all shared skills. This skill keeps your local copy fresh by:

1. Cloning the repo (if not already present)
2. Pulling latest changes (if already cloned)
3. Showing what's new/changed
4. Verifying all skills are available
5. Displaying current version

Use this skill to ensure you're always using the latest, most improved versions of shared skills.

---

## How It Works

### For Claude (Local)

```
"Can you use sync-skills to update the skills?"

sync-skills will:
1. Check if ~/claude-skills exists
2. If not: Clone https://github.com/YOUR_ACCOUNT/claude-skills
3. If yes: Pull latest from GitHub
4. Show changelog (new skills, updates, fixes)
5. List all available skills
6. Display version and last update date
```

### For Hermes (or Remote Agents)

```
(You brief Hermes)
"Clone the skills repo and use sync-skills to keep it fresh.
Skills repo: https://github.com/YOUR_ACCOUNT/claude-skills"

(Hermes)
1. Clones the repo to its workspace
2. Uses sync-skills to pull updates before using other skills
3. Has access to all available skills
```

---

## Before Using Other Skills

**Always run sync-skills first:**

```
Step 1: "Use sync-skills to update"
         ↓
Step 2: "Now use roomsense-wishlist-first to check backlog"
         ↓
Step 3: "Use grow-up to analyze this problem"
```

This ensures you have:
- ✅ Latest skill definitions
- ✅ Recent improvements and fixes
- ✅ All dependencies resolved
- ✅ Version compatibility verified

---

## What Gets Updated

### Skills (New & Improved)

Each skill has a `version` in the MANIFEST:
- New skill added → `version 1.0.0`
- Skill improved → `version 1.1.0`, `1.2.0`, etc.
- Breaking change → `version 2.0.0`

sync-skills shows:
```
✅ roomsense-wishlist-first: 1.0.0 (no changes)
✅ grow-up: 1.1.0 (improved root cause analysis)
✅ sync-skills: 1.0.0 (current)

Latest repo version: 1.2.0 (2 skills updated since your last sync)
```

### Changelog

Shows what changed since your last sync:
```
2026-07-22: Added sync-skills skill
2026-07-21: Improved grow-up RCA flexibility
2026-07-20: Created roomsense-wishlist-first
```

---

## Usage Examples

### Example 1: Before Starting RoomSense Work

```
You: "Hermes, I need backend work for Strategy 2. First, use sync-skills."

Hermes:
1. Clones skills repo
2. Runs sync-skills
3. Sees roomsense-wishlist-first skill
4. Reads SKILL.md for pre-work checklist
5. Adds items to wishlist with @H ownership
6. Proceeds with backend work
```

### Example 2: After Problem Happens

```
You: "Something broke. Use sync-skills, then use grow-up to analyze."

Claude:
1. Runs sync-skills → shows latest grow-up version
2. Uses grow-up skill to analyze root cause
3. Creates/updates skills to prevent recurrence
4. Commits learning with reference to grow-up v1.2.0
```

### Example 3: Staying Fresh Across Sessions

```
(First task)
You: "Claude, use sync-skills then start the work"
Claude: Clones repo, pulls latest

(Later, second task)
You: "Use sync-skills to check for updates"
Claude: Pulls latest (might be new skills or improvements)
Claude: Shows changelog
You: Have latest tools available
```

---

## Commands

### Explicit Update

```bash
# Claude can run this directly
cd ~/claude-skills && git pull origin main

# Or use the skill
"Use sync-skills to check for updates"
```

### Check Current Version

```bash
# Show what version you have
cd ~/claude-skills && cat MANIFEST.json | grep '"version"'

# Show when it was last updated
cd ~/claude-skills && git log -1 --format="%ai" README.md
```

### View Changelog

```bash
# Show recent commits to the repo
cd ~/claude-skills && git log --oneline -10
```

---

## Initial Setup (One Time)

### For Claude

```bash
# Clone the repo once
git clone https://github.com/YOUR_ACCOUNT/claude-skills ~/claude-skills

# From then on, sync-skills keeps it fresh
# Or manually: cd ~/claude-skills && git pull
```

### For Hermes (When You Brief Them)

```
(You to Hermes)
"Clone the skills repo: https://github.com/YOUR_ACCOUNT/claude-skills
Then use sync-skills before you start working."

(Hermes)
1. Clones the repo
2. Runs sync-skills
3. Ready to use all skills
```

---

## What Sync-Skills Checks

✅ **Repo connectivity:** Can reach GitHub  
✅ **Latest commits:** Pulls all new changes  
✅ **Skill integrity:** All skills have valid SKILL.md  
✅ **MANIFEST validity:** skill registry is valid JSON  
✅ **Version alignment:** Local skills match MANIFEST versions  
✅ **Completeness:** All skills listed in MANIFEST exist on disk  

If anything is missing or broken, sync-skills reports it:

```
⚠️ WARNING: skill "example-skill" listed in MANIFEST but not found on disk
⚠️ WARNING: MANIFEST.json is invalid JSON
⚠️ ERROR: Cannot reach GitHub repo (check internet connection)
```

---

## Versioning Strategy

Skills use semantic versioning:

| Version | Meaning | When |
|---------|---------|------|
| `1.0.0` | Initial release | Skill is new and stable |
| `1.1.0` | Minor improvement | Bug fix, small enhancement, no breaking changes |
| `1.2.0` | Another improvement | Additional fix or feature, still compatible |
| `2.0.0` | Major overhaul | Breaking changes, old syntax no longer works |

**Your local copy shows:**
```
roomsense-wishlist-first: 1.0.0
grow-up: 1.1.0 (latest 1.2.0 available)
```

If you're on 1.1.0 and 1.2.0 is available, sync-skills will:
1. Show the upgrade is available
2. Pull the new version
3. Show what changed
4. You're now on 1.2.0

---

## Troubleshooting

### "Can't reach GitHub"

Check internet connection and repo URL:
```bash
cd ~/claude-skills && git remote -v
# Should show: origin https://github.com/YOUR_ACCOUNT/claude-skills (fetch/push)
```

### "Local changes would be overwritten"

If you've modified a skill locally and can't pull:
```bash
cd ~/claude-skills && git stash
# Then sync-skills will pull cleanly
```

### "Skills seem broken"

Verify all skills are intact:
```bash
cd ~/claude-skills && ls -la
# Should see: roomsense-wishlist-first/, grow-up/, sync-skills/, etc.

# Check MANIFEST
cat MANIFEST.json | jq .skills
```

---

## When to Sync

**Always before:**
- Using another skill (ensure latest version)
- Sharing a skill with another agent
- Asking an agent to use a skill

**Good practice:**
- At the start of each work session
- After long pause (days/weeks)
- Before critical work
- If you suspect a skill was updated

**Automated:**
- Some agents might auto-sync before every skill invocation

---

## Summary

**The mantra:** Keep your tools sharp. Sync before you use.

```
Use sync-skills before every other skill invocation:

sync-skills → [other skills] → actual work

This ensures:
✅ Latest skill definitions
✅ Recent improvements & bug fixes
✅ All dependencies available
✅ Version compatibility verified
```

---

## References

- **Repo:** https://github.com/YOUR_ACCOUNT/claude-skills
- **MANIFEST:** See `MANIFEST.json` for skill registry
- **Version history:** See `MANIFEST.json` versions section
- **Changelog:** Run `git log` in skills repo

---

**Last updated:** 2026-07-22  
**Current version:** 1.0.0
