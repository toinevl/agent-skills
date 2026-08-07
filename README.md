# Agent Skills Library

Reusable skills for Claude, Hermes, and other agents. Each skill automates a specific workflow or pattern.

**Repository:** `https://github.com/toinevl/agent-skills`  
**Access:** Clone and point agents to this directory  
**Update:** Use the `tvv-sync-skills` skill to keep your local copy fresh

---

## Available Skills

| Skill | Purpose | Use When |
|-------|---------|----------|
| **tvv-roomsense-wishlist-first** | Enforce wishlist-first discipline in RoomSense | Starting work, reviewing commits, coordinating lanes (@C, @H, @O) |
| **tvv-grow-up** | Turn problems into permanent improvements via root cause analysis | Something breaks, a mistake happens, a pattern fails, you want to prevent recurrence |
| **tvv-sync-skills** | Keep your local skills repo updated from GitHub | Before using other skills, want latest versions |
| **tvv-verify-external-target** | Verify against the live/public source before implementing external-alignment work | Told to match/mirror/align to a brand, product, competitor, or spec — especially if the "official" source is blocked |
| **tvv-infra-script-authoring** | Author infrastructure scripts for platforms you can't test against | Writing bash/Python automation for Proxmox, Azure, AWS, K8s without a live test instance |
| **tvv-ops-infra** | Review Bicep/ARM IaC for correctness, security, and cost | Reviewing infra changes before merging |
| **tvv-ops-pipeline** | Review GitHub Actions CI/CD pipelines | Reviewing workflow changes before merging |
| **tvv-arch-api-design** | Design and document a REST API contract before implementation | Starting a new endpoint |
| **tvv-arch-adr** | Draft an Architecture Decision Record | Documenting a significant architectural decision |
| **tvv-arch-spec** | Generate a technical spec for a new feature | Before implementation starts on a non-trivial feature |
| **tvv-check-release** | Pre-release checklist (env vars, secrets, storage, deploy verification) | Before deploying to production |
| **tvv-azure-resource-hygiene** | Decommission replaced resources after Azure migrations | Migrating Azure resources, changing hosting model, deploying Bicep changes |
| **tvv-tue-business-case-dashboard** | Build/update an interactive HTML business case dashboard for a TU/e initiative from real pptx/docx source decks | Asked to create, build, or refresh a TU/e business case document or dashboard |
| **tvv-escalate-blocked-fetch** | Escalate to a real Playwright browser session when a raw HTTP fetch (WebFetch/web_url_read/curl) is blocked | A fetch returns 403/bot-detection/CAPTCHA and you're about to declare the site unreachable or switch to a substitute source |

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
before Claude can invoke them. The `tvv-sync-skills` skill does both steps.

### For Hermes (or any agent)

When briefing Hermes, include:

```
Skills repo: https://github.com/toinevl/agent-skills

Use the tvv-roomsense-wishlist-first skill to check if work is tracked on wishlist.
Use the tvv-grow-up skill to analyze this problem and prevent recurrence.
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
use the tvv-roomsense-wishlist-first skill to add items to the wishlist. 
Skills repo: https://github.com/toinevl/agent-skills"

(Hermes)
1. Clones https://github.com/toinevl/agent-skills
2. Reads ~/AI-Projects/agent-skills/tvv-roomsense-wishlist-first/SKILL.md
3. Adds items to wishlist with @H (Hermes) lane ownership
4. Commits with proper references
```

### Example: Using tvv-grow-up for root cause analysis

```
"I made a mistake: [describe]. Root cause: [why].
Can you use the tvv-grow-up skill to analyze this and prevent it?"

tvv-grow-up skill will:
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
tvv-skill-name/
├── SKILL.md              ← Main instructions + metadata frontmatter (read by agents)
├── evals/evals.json      ← Test cases
├── README.md             ← (Optional) Skill-specific docs
└── references/           ← (Optional) Supporting docs
    └── *.md
```

---

## Adding New Skills

Every skill is prefixed `tvv-` so it never collides with a builtin skill of the
same name — the ambiguity that led to two skills here duplicating builtins.

1. Create skill directory: `mkdir tvv-skill-name`
2. Add `SKILL.md` with full frontmatter. Every field is required; `name` must
   match the directory exactly, and `version` must be `MAJOR.MINOR.PATCH`:
   ```yaml
   ---
   name: tvv-skill-name
   description: 'Clear description of when to use this skill. Quote the whole value if it contains a colon.'
   version: 1.0.0
   category: architecture | ops | process | meta | maintenance
   scope: [all]
   triggers: [phrase that should invoke this, another phrase]
   status: stable
   created: '2026-07-29'
   last_updated: '2026-07-29'
   ---
   ```
3. Add `evals/evals.json` with test cases
4. Add `README.md` explaining the skill
5. Regenerate the manifest: `tools/build-manifest.py`
6. Commit: `git add tvv-skill-name/ MANIFEST.json && git commit -m "skill: add tvv-skill-name"`
7. Push to GitHub

### Versioning

Bump `version` in the skill's own `SKILL.md` when you change it — patch for
wording, minor for new capability, major for a change that alters how the skill
behaves for existing callers. Then rerun `tools/build-manifest.py`.

The repo-level `version` in `manifest.meta.json` is separate: bump it when
skills are added or removed, and add a matching entry under `versions`.

---

## Keeping Skills Fresh

Use the **tvv-sync-skills** skill to pull latest from GitHub:

```
"Can you use the tvv-sync-skills skill to update the skills repo?"

tvv-sync-skills will:
1. Pull latest from https://github.com/toinevl/agent-skills
2. Show what changed
3. Update your local copy
4. Confirm all skills are ready
```

---

## Manifest

`MANIFEST.json` is the machine-readable registry agents use to discover skills.

**It is generated — do not edit it by hand.** Its `skills[]` array is built from
the `SKILL.md` frontmatter of every `tvv-*` directory:

```bash
tools/build-manifest.py           # regenerate after editing any SKILL.md
tools/build-manifest.py --check   # verify it is current (runs in CI)
```

Repo-level fields — version history and the record of removed skills — live in
`manifest.meta.json` and are maintained by hand.

CI runs `--check` on every PR, so the manifest cannot drift from the skills the
way it did before v2.0.0. The check also rejects frontmatter that is not valid
YAML, a directory whose name does not match its `name:` field, a missing
`tvv-` prefix, and a `version` that is not semver.

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

**Last updated:** 2026-08-07  
**Total skills:** 14 (tvv-roomsense-wishlist-first, tvv-grow-up, tvv-sync-skills, tvv-verify-external-target,
tvv-infra-script-authoring, tvv-ops-infra, tvv-ops-pipeline, tvv-arch-api-design, tvv-arch-adr, tvv-arch-spec,
tvv-check-release, tvv-azure-resource-hygiene, tvv-tue-business-case-dashboard, tvv-escalate-blocked-fetch)

> `azure-functions-deploy` and `azure-swa-deploy` were removed in `244023a` — both are
> covered by Claude's builtin skills. This repo keeps only skills that have no builtin
> equivalent.
