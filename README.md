# Agent Skills Library

Reusable skills for Claude, Hermes, and other agents. Each skill automates a specific workflow or pattern.

**Repository:** `https://github.com/toinevl/agent-skills`  
**Access:** Clone and point agents to this directory  
**Update:** Use the `sync-skills` skill to keep your local copy fresh

---

## Available Skills

| Skill | Purpose | Use When |
|-------|---------|----------|
| **roomsense-wishlist-first** | Enforce wishlist-first discipline in RoomSense | Starting work, reviewing commits, coordinating lanes (@C, @H, @O) |
| **grow-up** | Turn problems into permanent improvements via root cause analysis | Something breaks, a mistake happens, a pattern fails, you want to prevent recurrence |
| **sync-skills** | Keep your local skills repo updated from GitHub | Before using other skills, want latest versions |
| **verify-external-target** | Verify against the live/public source before implementing external-alignment work | Told to match/mirror/align to a brand, product, competitor, or spec — especially if the "official" source is blocked |
| **infra-script-authoring** | Author infrastructure scripts for platforms you can't test against | Writing bash/Python automation for Proxmox, Azure, AWS, K8s without a live test instance |
| **ops-infra** | Review Bicep/ARM IaC for correctness, security, and cost | Reviewing infra changes before merging |
| **ops-pipeline** | Review GitHub Actions CI/CD pipelines | Reviewing workflow changes before merging |
| **arch-api-design** | Design and document a REST API contract before implementation | Starting a new endpoint |
| **arch-adr** | Draft an Architecture Decision Record | Documenting a significant architectural decision |
| **arch-spec** | Generate a technical spec for a new feature | Before implementation starts on a non-trivial feature |
| **check-release** | Pre-release checklist (env vars, secrets, storage, deploy verification) | Before deploying to production |

---

## Quick Start

### For Claude (Local)

```bash
# Clone the repo (canonical location)
git clone https://github.com/toinevl/agent-skills ~/AI-Projects/agent-skills

# Install the skills so Claude can invoke them
cp -r ~/AI-Projects/agent-skills/<skill-name> ~/.claude/skills/<skill-name>
```

Cloning alone is not enough — skills must be copied into `~/.claude/skills/`
before Claude can invoke them. The `sync-skills` skill does both steps.

### For Hermes (or any agent)

When briefing Hermes, include:

```
Skills repo: https://github.com/toinevl/agent-skills

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
Skills repo: https://github.com/toinevl/agent-skills"

(Hermes)
1. Clones https://github.com/toinevl/agent-skills
2. Reads ~/AI-Projects/agent-skills/roomsense-wishlist-first/SKILL.md
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
1. Pull latest from https://github.com/toinevl/agent-skills
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

**Last updated:** 2026-07-29  
**Total skills:** 11 (roomsense-wishlist-first, grow-up, sync-skills, verify-external-target,
infra-script-authoring, ops-infra, ops-pipeline, arch-api-design, arch-adr, arch-spec,
check-release)

> `azure-functions-deploy` and `azure-swa-deploy` were removed in `244023a` — both are
> covered by Claude's builtin skills. This repo keeps only skills that have no builtin
> equivalent.
