# Proxmox VE: Authoring Checklist for `qm` Scripts

Use this reference when authoring bash automation that calls Proxmox VE's
`qm`, `pvesm`, and `pveversion` CLIs. Each item comes from a bug that shipped
to production because a value or flag was guessed rather than verified.

## Reference script

The enterprise-grade reference script embedding all of these rules:
`/home/toine/AI-Projects/projects/playground/proxmox-arch-template/proxmox-arch-template.sh`
v1.0.0.

## 1. `pveversion` — bare command only, no `--version`

`pveversion` takes no flags. There is no `--version`, no `-v`, no `-V`.
Calling it with `--version` returns an error/broken regex capture.

```bash
# CORRECT
PVE_VER=$(pveversion | grep -oP 'pve-manager/\K[0-9]+' | head -1 || echo "0")
```

Always test the regex against the real output:
```
pve-manager/8.4.17/abcdef (running kernel: 6.8.12-13-pve)
```

## 2. Storage content types — don't use `pvesm status --content`

`pvesm status --storage X --content` filters results BY content type; it
does NOT list which content types storage X supports. On a fresh Proxmox
install, it returns empty for `local-lvm --content` even though `images` is
valid.

The authoritative source is `/etc/pve/storage.cfg`.

```bash
get_storage_content() {
  local storage="$1"
  awk -v s="$storage" '
    /^[a-z]/ { section=$0; sub(/^[^:]+:/, "", section); gsub(/[ \t]/, "", section); in_sec=(section==s); next }
    in_sec && /^[ \t]+content/ { gsub(/[ \t]/, "", $0); sub(/^content/, "", $0); print; exit }
  ' /etc/pve/storage.cfg 2>/dev/null
}
```

## 3. `local` storage lacks `snippets` by default — auto-fix, don't error

Fresh PVE installs: `local` has `iso,vztmpl,backup` only. Cloud-init custom
user-data requires `snippets`. Your script should append `snippets`
non-destructively instead of hard-failing.

```bash
LOCAL_CONTENT=$(get_storage_content "$SNIPPET_STORAGE")
if [[ "$LOCAL_CONTENT" != *"snippets"* ]]; then
  NEW_CONTENT="${LOCAL_CONTENT:+$LOCAL_CONTENT,}snippets"
  pvesm set "$SNIPPET_STORAGE" --content "$NEW_CONTENT"
fi
```

## 4. `efidisk0` — `efitype` enum is `2m` or `4m`, NOT `4k`

PVE rejects `efitype=4k` with `value does not have a value in the enumeration
'2m, 4m'`. The values are mebibytes, not kilobytes.

```bash
--efidisk0 "${STORAGE}:1,efitype=4m,pre-enrolled-keys=0"
```

## 5. `bios`, `machine`, `ostype` — verify enums in docs

Write these values only after checking PVE API docs for that PVE version.

| Flag | Valid values | Source |
|------|-------------|--------|
| `--bios` | `seabios`, `ovmf` | PVE API reference |
| `--machine` | `pc`, `q35`, ... | PVE API reference |
| `--ostype` | `l24` (Linux 2.4), `l26` (Linux 2.6+), ... | PVE API reference |
| `--scsihw` | `virtio-scsi-pci`, ... | PVE API reference |
| `--vga` | `serial0`, `std`, `vmware`, ... | PVE API reference |

## 6. `cicustom` format — use the resolved snippet storage, not hardcoded

Cloud-init custom user-data files live on a snippet storage. The storage
name may be `local` but its path may differ if the user customized locations.

```bash
# Resolve the actual snippet directory
STORAGE_PATH=$(awk -v s="$SNIPPET_STORAGE" '
  /^[a-z]/ { sec=$0; sub(/^[^:]+:/, "", sec); gsub(/[ \t]/, "", sec); in=(sec==s); next }
  in && /^[ \t]+path/ { gsub(/[ \t]/, "", $0); sub(/^path/, "", $0); print; exit }
' /etc/pve/storage.cfg)

SNIPPET_DIR="${STORAGE_PATH:-/var/lib/vz}/snippets"
mkdir -p "$SNIPPET_DIR"

# Reference via the STORAGE variable, never hardcoded "local"
qm set "$VMID" --cicustom "user=${SNIPPET_STORAGE}:snippets/arch-template-${VMID}-user.yaml"
```

## 7. `importdisk` volume naming — don't assume

Creating an EFI disk first can consume `disk-0`, shifting the imported disk
volumes. Parse the actual volume name from `qm importdisk` output.

```bash
IMPORT_OUTPUT=$(qm importdisk "$VMID" "$IMAGE_FILE" "$STORAGE" --format raw 2>&1)
VOL_KEY=$(echo "$IMPORT_OUTPUT" | grep -oP "unused0:\K[^\s']+" || echo "")
if [[ -n "$VOL_KEY" ]]; then
  qm set "$VMID" --scsi0 "${VOL_KEY},discard=on,ssd=1"
else
  qm set "$VMID" --scsi0 "${STORAGE}:vm-${VMID}-disk-1,discard=on,ssd=1"
fi
```

## 8. QA discipline: verify correctness, not just presence

A QA harness that greps for a pattern you wrote can only confirm the value
is present — not that it is correct. Before writing, look up the actual
values in PVE API docs. When you discover a wrong value:

1. Fix the script
2. Add a test assertion for the CORRECT value
3. Record the lesson in this file or skill

## Verification checklist

Before running against the Proxmox host:

- [ ] `bash -n <script>` — syntax check
- [ ] No `YOUR_ACCOUNT`, `TODO`, `CHANGEME` strings
- [ ] `pveversion` call has no flags
- [ ] `/etc/pve/storage.cfg` parser tests against real output
- [ ] `efitype=` uses only `2m` or `4m`
- [ ] `cicustom` uses a variable, not `local:snippets/...`
- [ ] `importdisk` volume name is parsed, not hardcoded
- [ ] Script auto-fixes missing `snippets` content type
- [ ] Script passes `--dry-run` on a non-PVE test host (at least `-u` catch)
