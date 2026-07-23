---
name: infra-script-authoring
description: "Author infrastructure scripts for platforms you can't test against — verify API values from docs, never guess."
version: 1.0.0
tags: [devops, infrastructure, proxmox, azure, aws, bash, scripting, qa]
platforms: [linux]
metadata:
  hermes:
    tags: [devops, infrastructure, scripting, qa]
---

# Infrastructure Script Authoring (for platforms you can't test against)

## When to use
Writing bash/Python automation for infrastructure platforms (Proxmox, VMware, AWS CLI, Azure CLI, Kubernetes) when you don't have a live instance to test against.

## Core principle

**Never invent API parameter values or CLI flags.** Every enum value, flag name, and parameter format must come from official documentation or a working example you've verified. A script with perfect error handling that calls a non-existent flag is worse than a naive script that works.

## The lesson that created this skill

Three bugs shipped to a Proxmox host in successive runs:
1. `pveversion --version` — flag doesn't exist (correct: bare `pveversion`)
2. `pvesm status --content` — filters BY content type, doesn't list types (correct: read `/etc/pve/storage.cfg`)
3. `efitype=4k` — invalid enum (correct: `2m` or `4m`)

Root cause: the script was written from memory/plausible reasoning, not from docs. The grep-based QA checked "does my script contain the string I wrote" — never "is this value correct for the target platform."

## Pre-write checklist (mandatory)

1. **Web-search official docs** for every CLI command and API parameter before writing
2. **Verify enum values** — don't guess. If docs say `2m, 4m`, don't write `4k`
3. **Verify CLI flags** — don't assume `--version` exists on every tool. Check `man` or `--help` from docs
4. **Verify output formats** — know what the command actually prints before writing a parser
5. **Verify default states** — know what's enabled by default on a fresh install
6. **Read community examples** — forum posts showing working commands catch what docs miss
7. **Verify external URLs before writing them** — do a HEAD request or browse the mirror directory; never hardcode a download URL you haven't confirmed returns 200

## Anti-pattern: hardcoded download URLs

Scripts that download ISOs, packages, or binaries often hardcode a URL assumed to be stable. This always eventually breaks — mirrors reorganize, projects rename releases, "latest" symlinks are never created.

**Bad:**
```bash
wget https://mirror.cachyos.org/iso/cachyos-linux-stable.iso  # 404 — path never existed
```

**Good:**
```bash
# Browse the mirror directory first, then write the versioned path you actually confirmed
# https://mirror.cachyos.org/ISO/desktop/ → dated subdirs → e.g. 260628/cachyos-desktop-linux-260628.iso
wget https://mirror.cachyos.org/ISO/desktop/260628/cachyos-desktop-linux-260628.iso
```

**Better:** write the script to browse the directory and pick the latest entry dynamically, so it doesn't need updating with each release.

```bash
# Find latest dated subdir and download from it
BASE="https://mirror.cachyos.org/ISO/desktop"
LATEST=$(curl -s "$BASE/" | grep -oP 'href="\K[0-9]+(?=/)' | sort -n | tail -1)
ISO=$(curl -s "$BASE/$LATEST/" | grep -oP 'href="\K[^"]+\.iso' | head -1)
wget "$BASE/$LATEST/$ISO"
```

**Verification rule:** before writing any URL into a script or README, run `curl -sI <url> | head -1` and confirm it returns `200` or `302`, not `404`.

---

## Anti-pattern: self-referential QA

Grep-based verification that checks "does my script contain the string I wrote" is NOT verification. It confirms you typed something, not that it's correct.

### Bad QA
```bash
check "has efitype" grep -q 'efitype' "$SCRIPT"
# Passes whether the value is 4k, 4m, or banana
```

### Good QA
```bash
# Verify the value is in the documented enum set
check "efitype is valid" grep -qE 'efitype=(2m|4m)' "$SCRIPT"
```

### Even better: simulate
```bash
# Feed mock command output into your parser to verify it works
echo 'pve-manager/8.4.17/abc (running kernel: 6.8)' | grep -oP 'pve-manager/\K[0-9]+'
# Verify output is "8"
```

## Verification strategy for untestable platforms

1. **Web-search the API docs** for every parameter value BEFORE writing the script
2. **Cross-reference** 2+ sources (official docs + community examples) for non-obvious values
3. **Simulate** parser logic against documented output formats
4. **Dry-run first** — always offer `--dry-run` and tell the user to try it before the real run
5. **Fail fast with clear errors** — when something IS wrong, the user gets actionable feedback

## Process for fixing bugs found at runtime

1. Acknowledge the bug honestly (don't minimize it)
2. Search docs for the correct value/command
3. Fix the script
4. Add a verification check for the SPECIFIC correct value (not just presence)
5. Update the platform reference below if reusable

---

## Platform reference: Proxmox VE 8.x (verified values)

### pveversion
- Correct: `pveversion` (no flags)
- NOT `pveversion --version` (that flag doesn't exist)
- Output format: `pve-manager/8.4.17/c8c39014680186a7 (running kernel: 6.8.12-13-pve)`
- Extract major: `pveversion | grep -oP 'pve-manager/\K[0-9]+'`

### pvesm status
- `pvesm status` — lists all storage
- `pvesm status --storage NAME` — checks storage exists
- `--content` FILTERS by content type — does NOT list supported types
- To get content types: awk `/etc/pve/storage.cfg`
- storage.cfg format:
  ```
  lvmthin: local-lvm
          thinpool data
          vgname pve
          content images,rootdir
  ```

### qm create — efidisk
- `efitype` accepts: `2m` or `4m` (NOT `4k`)
- Format: `--efidisk0 STORAGE:1,efitype=4m,pre-enrolled-keys=0`

### qm importdisk
- Output contains volume key: `unused0:STORAGE:vm-VMID-disk-N`
- Disk numbering is NOT predictable (efidisk may take disk-0)
- Parse the output to get the real volume key, don't hardcode

### Snippets storage
- `local` storage ships WITHOUT snippets content type by default
- Auto-fix: `pvesm set local --content EXISTING,snippets` (additive, safe)
- Snippet directory comes from storage.cfg `path` property
- Default path: `/var/lib/vz/snippets`

### Disk space check
- `pvesm status --storage NAME --output avail` — returns bytes available

---

## Platform reference: Azure Functions (verified values)

### Consumption (Y1/Dynamic) plan deploy
- Deploy via `func azure functionapp publish NAME --node` (NOT `Azure/functions-action@v1`)
- Kudu zipdeploy produces 503 "Function host is not running" on Consumption Linux
- Deploy package must be self-contained (minimal package.json, no `workspace:*` deps, `npm install` inside)
- `.funcignore` must NOT exclude `dist` directory
- `WEBSITE_RUN_FROM_PACKAGE` can block zip deploys — clear it if stuck

### Flex Consumption CORS
- Flex Consumption short-circuits CORS preflights (OPTIONS) at the Kestrel front-end
- Returns empty 204 before function code runs — function-level CORS never executes
- Fix: use Consumption (Y1/Dynamic) plan instead

### Azure Table Storage
- No array type — serialize arrays as JSON strings, deserialize on read
- Entity properties are limited to primitive EDM types
