---
name: sync-skills
description: "Keep your local skills repo updated from GitHub. Pull latest, copy new skills to the local install directory, show what changed, verify all skills present."
version: 1.3.0
tags: [skills, sync, maintenance]
platforms: [linux, macos]
---

# sync-skills: Keep Skills Repo and Local Install Updated

## Purpose

Pull the latest skills from https://github.com/toinevl/agent-skills,
install them into your local skills directory, and report what changed.

**Critical:** pulling the repo without copying to the local skills directory means the agent can't use the new skills. Both steps are required.

## When to use

- At the start of a work session
- Before using other skills from the repo
- After someone else pushed updates to the repo

## Execution

The repo lives at `~/AI-Projects/agent-skills` on every machine. Clone it there if missing:

```bash
[ -d ~/AI-Projects/agent-skills ] || git clone https://github.com/toinevl/agent-skills ~/AI-Projects/agent-skills
```

```bash
# 1. Pull latest from the repo
cd ~/AI-Projects/agent-skills
git pull origin main

# 2. Show recent changes
git log --oneline -5

# 3. List all skills in the repo
find . -maxdepth 2 -name 'SKILL.md' | sort
```

## Install new/updated skills

After pulling, copy any new or changed skill folders to your local skills directory.

**For Claude** (local skills at `~/.claude/skills/`):
```bash
mkdir -p ~/.claude/skills
cp -r ~/AI-Projects/agent-skills/<skill-name> ~/.claude/skills/<skill-name>
```

**For Hermes** (local skills at `~/.hermes/profiles/glm/skills/`):
```bash
# devops category skills
cp -r ~/AI-Projects/agent-skills/<skill-name> ~/.hermes/profiles/glm/skills/devops/<skill-name>
# top-level skills
cp -r ~/AI-Projects/agent-skills/<skill-name> ~/.hermes/profiles/glm/skills/<skill-name>
```

## Skill-to-category mapping (Hermes)

| Skill | Category | Rationale |
|-------|----------|-----------|
| infra-script-authoring | devops | Infrastructure automation patterns |
| ops-infra | devops | IaC review |
| ops-pipeline | devops | CI/CD review |
| arch-api-design | (top-level) | Architecture, not devops |
| arch-adr | (top-level) | Architecture |
| arch-spec | (top-level) | Architecture |
| check-release | (top-level) | Process |
| grow-up | (top-level) | General workflow |
| roomsense-wishlist-first | (top-level) | Project-specific |
| sync-skills | (top-level) | Meta-skill |

## Verification

After sync, verify all skills load:
- Each skill has a valid `SKILL.md` with YAML frontmatter
- No placeholder URLs (`YOUR_ACCOUNT`) remain
- MANIFEST version: `python3 -c "import json; d=json.load(open('$HOME/AI-Projects/agent-skills/MANIFEST.json')); print(d['version'], d['last_updated'])"`
- Every skill directory in the repo appears in `MANIFEST.json`, and every manifest
  `path` resolves to a real directory — the two drift apart when a skill is deleted
  without updating the manifest

**Last updated:** 2026-07-29
**Current version:** 1.3.0
