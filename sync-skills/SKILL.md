---
name: sync-skills
description: Keep your local skills repo and ~/.claude/skills/ updated from GitHub. Pulls latest from toinevl/agent-skills, then copies any new or updated skill folders into ~/.claude/skills/ so they are immediately available. Run at the start of every session or whenever the user says "sync skills". Triggers on: "sync skills", "update skills", "pull latest skills".
---

# sync-skills: Keep Skills Repo and Local Install Updated

## Repo

`https://github.com/toinevl/agent-skills` → cloned at `~/agent-skills`  
Local install target: `~/.claude/skills/`

**Both must be updated.** Pulling the repo without copying to `~/.claude/skills/` means the agent can't use the new skills. This is the critical step the old version of this skill omitted.

---

## Steps — run every time

**Step 1: Pull from GitHub**

```bash
cd ~/agent-skills && git pull origin main 2>&1
```

Note the output — `Already up to date` means nothing to do. Otherwise the output shows exactly which files changed.

**Step 2: Identify new or updated skill folders**

A skill folder is any directory containing a `SKILL.md`. Compare what pulled against what's already in `~/.claude/skills/`:

```bash
# List skill folders in the repo
ls ~/agent-skills/*/SKILL.md | xargs -I{} dirname {} | xargs -I{} basename {}

# List what's already installed
ls ~/.claude/skills/
```

**Step 3: Copy new/updated skills to `~/.claude/skills/`**

For each skill folder that is new or was modified in the pull:

```bash
cp -r ~/agent-skills/<skill-name> ~/.claude/skills/<skill-name>
```

Do this for every changed skill — `cp -r` overwrites cleanly.

**Step 4: Report**

Tell the user:
- What was pulled (git output)
- Which skills were copied to `~/.claude/skills/`
- Current MANIFEST version (`cat ~/agent-skills/MANIFEST.json | grep '"version"' | head -1`)

---

## Initial setup (if `~/agent-skills` doesn't exist)

```bash
git clone https://github.com/toinevl/agent-skills ~/agent-skills
```

Then run Steps 2–4 above to install all skills.

---

## Troubleshooting

**"Local changes would be overwritten"**
```bash
cd ~/agent-skills && git stash && git pull origin main
```

**Skill not showing up after copy**  
Verify the folder contains `SKILL.md`:
```bash
ls ~/.claude/skills/<skill-name>/
```

**Check current MANIFEST version**
```bash
cat ~/agent-skills/MANIFEST.json | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['version'], d['last_updated'])"
```

---

**Last updated:** 2026-07-23  
**Current version:** 1.1.0
