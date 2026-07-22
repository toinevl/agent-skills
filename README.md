# Agent Skills Library

Reusable skills for Claude, Hermes, and other agents. Each skill automates a specific workflow or pattern.

**Repository:** `https://github.com/YOUR_ACCOUNT/agent-skills`  
**Access:** Clone and point agents to this directory  
**Update:** Use the `sync-skills` skill to keep your local copy fresh

---

## Available Skills

| Skill | Purpose | Use When |
|-------|---------|----------|
| **roomsense-wishlist-first** | Enforce wishlist-first discipline in RoomSense | Starting work, reviewing commits, coordinating lanes (@C, @H, @O) |
| **grow-up** | Turn problems into permanent improvements via root cause analysis | Something breaks, a mistake happens, a pattern fails, you want to prevent recurrence |
| **sync-skills** | Keep your local skills repo updated from GitHub | Before using other skills, want latest versions |

---

## Quick Start

### For Claude (Local)

```bash
# Clone the repo
git clone https://github.com/YOUR_ACCOUNT/agent-skills ~/agent-skills

# Use a skill
# The skill will be available to invoke in Claude Code
```

### For Hermes (or any agent)

When briefing Hermes, include:

```
Skills repo: https://github.com/YOUR_ACCOUNT/agent-skills

Use the roomsense-wishlist-first skill to check if work is tracked on wishlist.
Use the grow-up skill to analyze this problem and prevent recurrence.
```

Hermes will:
1. Clone the repo
2. Access skills from that directory
3. Invoke them as needed

---

## Using Skills

### Example: Hermes using a skill

```
(You brief Hermes)
"I need you to do the backend work for Strategy 2. Before you start, 
use the roomsense-wishlist-first skill to add items to the wishlist. 
Skills repo: https://github.com/YOUR_ACCOUNT/claude-skills"

(Hermes)
1. Clones https://github.com/YOUR_ACCOUNT/claude-skills
2. Reads ~/claude-skills/roomsense-wishlist-first/SKILL.md
3. Adds items to wishlist with @H (Hermes) lane ownership
4. Commits with proper references
```

### Example: Using grow-up for root cause analysis

```
"I made a mistake: [describe]. Root cause: [why].
Can you use the grow-up skill to analyze this and prevent it?"

grow-up skill will:
1. Ask clarifying questions
2. Do root cause analysis
3. Create/update skills or CLAUDE.md rules
4. Automate the fix
5. Commit the learning
```

---

## Skill Structure

Each skill directory contains:

```
skill-name/
├── SKILL.md              ← Main instructions (read by agents)
├── evals/evals.json      ← Test cases
├── README.md             ← (Optional) Skill-specific docs
└── references/           ← (Optional) Supporting docs
    └── *.md
```

---

## Adding New Skills

1. Create skill directory: `mkdir skill-name`
2. Add `SKILL.md` with frontmatter:
   ```yaml
   ---
   name: skill-name
   description: Clear description of when to use this skill
   ---
   ```
3. Add `evals/evals.json` with test cases
4. Add `README.md` explaining the skill
5. Commit: `git add skill-name/ && git commit -m "skill: add skill-name"`
6. Push to GitHub

---

## Keeping Skills Fresh

Use the **sync-skills** skill to pull latest from GitHub:

```
"Can you use the sync-skills skill to update the skills repo?"

sync-skills will:
1. Pull latest from https://github.com/YOUR_ACCOUNT/claude-skills
2. Show what changed
3. Update your local copy
4. Confirm all skills are ready
```

---

## Manifest

See `MANIFEST.json` for machine-readable skill registry (used by agents to discover skills).

---

## Contributing

When improving a skill:

1. Edit the skill locally
2. Test it
3. Commit: `git commit -m "skill: improve [name] - [what changed]"`
4. Push to GitHub
5. Agents will pull latest on next sync

---

## Questions?

- **For Claude (local):** I have access to skills in `~/.claude/skills/`
- **For Hermes (remote):** Provide the repo URL when briefing
- **For new skills:** Use the skill-creator to draft, test, and iterate

---

**Last updated:** 2026-07-22  
**Total skills:** 3 (roomsense-wishlist-first, grow-up, sync-skills)
