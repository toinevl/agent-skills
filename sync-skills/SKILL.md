---
name: sync-skills
description: "Keep your local skills repo updated from GitHub. Pull latest, show what changed, verify all skills present."
version: 1.1.0
tags: [skills, sync, maintenance]
platforms: [linux, macos]
---

# sync-skills: Keep Skills Repo Updated

## Purpose

Pull the latest skills from https://github.com/toinevl/agent-skills,
install them into the Hermes local skills directory, and report what changed.

## When to use

- At the start of a work session
- Before using other skills from the repo
- After someone else pushed updates to the repo

## What it does

1. Pull latest from the repo
2. Compare against installed local skills
3. Install new/updated skills
4. Show changelog of what changed
5. Verify all skills have valid SKILL.md

## Execution

```bash
# 1. Pull latest from the repo
cd /home/toine/AI-Projects/agent-skills
git pull origin main

# 2. Show recent changes
git log --oneline -10

# 3. List all skills in the repo
find . -maxdepth 2 -name 'SKILL.md' | sort

# 4. Sync each skill into Hermes local skills directory
#    - devops/ category skills go to ~/.hermes/profiles/glm/skills/devops/
#    - project-specific skills go to ~/.hermes/profiles/glm/skills/
```

## Skill-to-category mapping

| Skill | Category | Rationale |
|-------|----------|-----------|
| infra-script-authoring | devops | Infrastructure automation patterns |
| azure-functions-deploy | devops | Azure Functions deployment |
| azure-swa-deploy | devops | Static Web Apps deployment |
| grow-up | (top-level) | General workflow, not category-specific |
| roomsense-wishlist-first | (top-level) | Project-specific |
| sync-skills | (top-level) | Meta-skill |

## Verification

After sync, verify all skills load:
- `skills_list` shows all skills in their categories
- Each skill has a valid SKILL.md with YAML frontmatter
- No placeholder URLs (`YOUR_ACCOUNT`) remain
