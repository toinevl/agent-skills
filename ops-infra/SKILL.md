---
name: ops-infra
description: Review Infrastructure as Code (Bicep, ARM) for correctness, security, and cost. Use before merging any IaC change to Azure — catches misconfigured resources, missing CORS, secret leaks in templates, and cost surprises before they hit production. Triggers on: "review bicep", "check arm template", "review infra", "IaC review".
---

# ops-infra: Infrastructure as Code Review

Review the IaC in `$ARGUMENTS` (file, module, or directory).

**Step 1** — Read the IaC files and identify the resource types being created or modified.

## General Quality

- Resources named consistently and descriptively
- Tags present on all billable resources (environment, project, owner)
- No hardcoded values that belong in parameters or Key Vault references
- Parameters have descriptions and sensible defaults
- Outputs expose only what callers actually need

## Azure-Specific

- SKU/tier appropriate for environment (avoid Premium in dev, avoid Free in prod)
- Flex Consumption plan: `functionAppScaleLimit` set, always-ready instances configured if needed
- Static Web Apps: correct SKU (`Free` for static-only, `Standard` for linked APIs)
- Storage accounts: `allowBlobPublicAccess: false`, TLS 1.2 minimum, correct replication tier
- Function apps: `WEBSITE_RUN_FROM_PACKAGE: 1` set, `linuxFxVersion` pinned to exact runtime
- Key Vault references used for secrets — no plaintext connection strings in app settings
- CORS configured at platform level (`az functionapp cors`) for Functions, not only in app code
- Managed identity used where possible instead of connection strings

## Security

- No secrets, passwords, or keys hardcoded in the template
- Network access restricted where appropriate (IP allowlists, private endpoints)
- HTTPS-only enforced on all web-facing resources
- Diagnostic settings / logging enabled on critical resources

## Idempotency & Safety

- Resources that cannot be updated in-place identified (require delete/recreate — warn explicitly)
- `lock` resource present on production resource groups containing stateful data
- Deployment mode (`Complete` vs `Incremental`) appropriate — `Complete` only when intentional

## Cost

- No over-provisioned SKUs for the workload
- Consumption/serverless used where traffic is bursty or low
- Storage replication tier not over-specified (LRS sufficient for non-critical dev data)

---

**Output findings as a prioritized list: Critical > Major > Minor.** Include `file:line` references.
