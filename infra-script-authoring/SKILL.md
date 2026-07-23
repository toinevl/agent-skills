---
name: infra-script-authoring
description: "Author infrastructure scripts for platforms you can't test against — verify API values from docs, never guess, ship with enterprise safety."
version: 1.2.0
author: Hermes Agent
license: MIT
tags: [devops, infra, bash, proxmox, azure, api-values, verification]
metadata:
  hermes:
    related_skills: [enterprise-bash-scripts]
---

# Infrastructure Script Authoring

Write reliable infrastructure automation for platforms you cannot
test against locally (Proxmox, VMware, Azure CLI, Kubernetes, etc.).

## When to use

- Writing bash/CLI automation against a platform API you do not have
  live access to right now
- Authoring scripts meant to run on a remote host (bare metal,
  hypervisor, cloud tenant) where first-run failures are expensive
- Building on top of `enterprise-bash-scripts` (lockfiles, rollback,
  dry-run, retry): that skill covers the *structural* requirements.
  This skill covers the *correctness* requirements.

## Iron rule: verify API values before writing

Structural completeness (version flag, rollback, dry-run) is necessary
but not sufficient. Bugs from invented API values ship more often than
bugs from missing rollback code.

For every platform API value you write:

1. Look up the actual enum/syntax in official docs first
2. Copy the valid value verbatim — do not paraphrase
3. After the first failed run, add the correct value to a project-local
   checklist so the next author doesn't repeat the mistake

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

**Better:** write the script to browse the directory and pick the latest entry dynamically:

```bash
BASE="https://mirror.cachyos.org/ISO/desktop"
LATEST=$(curl -s "$BASE/" | grep -oP 'href="\K[0-9]+(?=/)' | sort -n | tail -1)
ISO=$(curl -s "$BASE/$LATEST/" | grep -oP 'href="\K[^"]+\.iso' | head -1)
wget "$BASE/$LATEST/$ISO"
```

**Verification rule:** before writing any URL into a script or README, run `curl -sI <url> | head -1` and confirm 200/302, not 404.

## Proxmox VE checklist

For Proxmox VE `qm` / `pvesm` / `pveversion` automation, use the
Proxmox-specific checklist:
`enterprise-bash-scripts` → `references/proxmox-ve-authoring-checklist.md`

Net rules:

- `pveversion` takes no flags
- `pvesm status --content` filters results BY content type, not the type
  a storage supports — parse `/etc/pve/storage.cfg` instead
- `local` storage ships WITHOUT `snippets`; append it non-destructively
- `efitype=4m` or `efitype=2m` — there is no `4k`
- `cicustom` must use the resolved snippet storage variable, never
  hardcoded `local`
- `importdisk` volume names may vary depending on disk order; parse the
  actual volume name from import output

## Terraform / IaC values

For Terraform, ARM/Bicep, and other IaC patterns:

1. Pull the valid argument names from the provider's own source or docs
2. Test against a canary run with `-var-file` before committing the template
3. Validate:

```bash
terraform validate
bicep build --file infra/main.bicep --stdout > /dev/null
```

## QA discipline: correctness vs presence

A grep-based verification (script contains string X) proves presence,
not correctness. Acceptable verification is one of:

1. Static validation: `bash -n`, `terraform validate`, `bicep build`
2. `--dry-run` on the target or closest equivalent host
3. Explicit assertion checks against known-good values written in the checklist reference

Recording the lesson in docs is not enough — add an explicit assertion.
A skill staying correct IS the discipline.

## Cross-reference with `enterprise-bash-scripts`

| Concern | Covered in |
|---------|------------|
| Structural requirements (versioning, rollback, dry-run) | enterprise-bash-scripts |
| Proxmox CLI values and enum traps | infra-script-authoring + Proxmox checklist |
| Azure CLI values (FunctionApp plan types, deploy methods) | azure-functions-deploy, azure-swa-deploy |

Load `enterprise-bash-scripts` for the structural skeleton, then load
this skill for platform-specific correctness constraints.

**Last updated:** 2026-07-23
**Current version:** 1.2.0
