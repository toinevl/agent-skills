---
name: tvv-sync-skills
description: Keep your local skills repo updated from GitHub. Pull latest, copy new skills to the local install directory, show what changed, verify all skills present.
version: 2.0.0
category: maintenance
scope: [all]
triggers: [before using skills, want latest versions, keep in sync]
status: stable
created: '2026-07-22'
last_updated: '2026-07-29'
tags: [skills, sync, maintenance]
platforms: [linux, macos]
---

# tvv-sync-skills: Keep Skills Repo and Local Install Updated

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
for d in ~/AI-Projects/agent-skills/tvv-*/; do
  n=$(basename "$d")
  rm -rf ~/.claude/skills/"$n"
  cp -r "$d" ~/.claude/skills/"$n"
done
```

**Prune pre-2.0.0 installs.** Every skill gained a `tvv-` prefix in v2.0.0. An
install directory left over from before the rename shadows nothing but does show
up as a duplicate in the skill picker — remove any unprefixed copy of a skill
that now exists as `tvv-*`:

```bash
for d in ~/AI-Projects/agent-skills/tvv-*/; do
  old=~/.claude/skills/$(basename "$d" | sed 's/^tvv-//')
  [ -d "$old" ] && echo "stale pre-2.0.0 install: $old"
done
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
| tvv-infra-script-authoring | devops | Infrastructure automation patterns |
| tvv-ops-infra | devops | IaC review |
| tvv-ops-pipeline | devops | CI/CD review |
| tvv-arch-api-design | (top-level) | Architecture, not devops |
| tvv-arch-adr | (top-level) | Architecture |
| tvv-arch-spec | (top-level) | Architecture |
| tvv-check-release | (top-level) | Process |
| tvv-grow-up | (top-level) | General workflow |
| tvv-roomsense-wishlist-first | (top-level) | Project-specific |
| tvv-sync-skills | (top-level) | Meta-skill |

## Verification

One command covers the structural checks — valid YAML frontmatter, required
fields, `tvv-` prefix, name/directory agreement, semver, and manifest freshness:

```bash
cd ~/AI-Projects/agent-skills && tools/build-manifest.py --check
```

If it reports the manifest is out of date, someone edited a `SKILL.md` without
regenerating. Run `tools/build-manifest.py` and commit the result.

Then confirm the install matches the repo:

```bash
diff <(ls ~/AI-Projects/agent-skills | grep '^tvv-') <(ls ~/.claude/skills | grep '^tvv-')
```

Remaining manual check:
- No placeholder URLs (`YOUR_ACCOUNT`) remain

**Last updated:** 2026-07-29
**Current version:** 2.0.0
