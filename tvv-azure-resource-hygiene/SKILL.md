---
name: tvv-azure-resource-hygiene
description: "Use after Azure migrations. Decommission replaced resources."
version: 1.0.0
category: devops
scope: [all]
triggers: [migrating azure resources, changing hosting model, replacing infra, deploying bicep changes, after architecture decision changes]
status: stable
created: '2026-07-31'
last_updated: '2026-07-31'
tags: [azure, infra, cleanup, cost, hygiene]
platforms: [linux, macos]
---

# tvv-azure-resource-hygiene: No Orphaned Resources After Migrations

## Purpose

Every infrastructure migration must end with decommissioning the resources it
replaces. This skill provides the checklist and commands to ensure nothing is
left behind.

## When to use

- Migrating an app between App Service plans or hosting models
- Changing storage accounts, key vaults, or function apps
- Any commit that deploys new Bicep/IaC that supersedes existing resources
- Architecture decision that changes where code runs (e.g., function API to server-side)
- After any session that created cloud resources "to try something"

## Pre-flight: Inventory Before Migration

Before starting any migration, snapshot what exists so you know what to clean up:

```bash
az resource list --resource-group <RG> -o table
```

Save this. After migration, diff against it. Anything that existed before but
is no longer referenced by the new architecture is an orphan.

## The Teardown Checklist

Run this AFTER the new infrastructure is deployed and verified healthy:

### 1. Identify what the migration replaced

For each resource that existed in the pre-flight inventory, ask:
  - Does the new architecture still reference it? (check app settings, CI,
    code imports, Key Vault references)
  - If NO, it is an orphan. Proceed to delete.

### 2. Verify no dependency before deleting

```bash
# Check if any app still references a storage account
az webapp config appsettings list --name <app> --resource-group <RG> \
  -o table | grep -i <storage-account-name>

# Check if any app still references a key vault
az webapp config appsettings list --name <app> --resource-group <RG> \
  -o table | grep -i <vault-name>

# Check what apps are on a plan before deleting the plan
az appservice plan show --name <plan> --resource-group <RG> \
  --query numberOfSites
```

### 3. Delete in dependency order

Resources have dependencies. Delete leaf resources first:

  1. Apps (Function Apps, Web Apps), before their App Service Plans
  2. App Service Plans, only after numberOfSites == 0
  3. App Insights, after the app consuming them is gone
  4. Storage accounts, after nothing references the connection string
  5. Key Vaults, after nothing references their secrets
  6. Static Web Apps, after custom domain DNS is repointed or accepted as gone

```bash
# Delete a web app / function app
az webapp delete --name <app> --resource-group <RG>

# Delete an app service plan (only if numberOfSites is 0 or null)
az appservice plan delete --name <plan> --resource-group <RG> --yes

# Delete App Insights
az monitor app-insights component delete --app <name> --resource-group <RG>

# Delete storage account
az storage account delete --name <name> --resource-group <RG> --yes

# Delete key vault (note: soft-delete lingers ~90 days, no cost)
az keyvault delete --name <name> --resource-group <RG>

# Delete static web app
az staticwebapp delete --name <name> --resource-group <RG> --yes
```

### 4. Post-teardown verification

```bash
# Confirm only intended resources remain
az resource list --resource-group <RG> -o table

# Confirm the live app is still healthy
curl -sI <app-url>/login | head -5
```

## Common Orphan Patterns (from experience)

| Migration | What gets orphaned | What to check |
|---|---|---|
| Static Web App to App Service | Old static site + its custom domain | Repository URL null? No CI deploying? Orphan. |
| External Function API to server-side | Function App + its plan + its storage + its KV + its App Insights | Code grep for function URL = 0 hits? Orphan. |
| Plan tier downgrade (S1 to F1) | Old plan if empty after move | numberOfSites == 0? Orphan. |
| Storage account migration | Old account + SAS tokens | No app setting references it? Orphan. |
| Created a plan to try something | Experimental plan in wrong region | 0 sites, never deployed to? Orphan. |

## Rules

1. **Decommission in the same PR/session.** If you deploy the replacement, you
   tear down the old. Do not leave it for "later."
2. **Dependents first.** Delete apps before plans, consumers before providers.
3. **Soft-delete awareness.** Key Vaults soft-delete for ~90 days. No cost, but
   the name can not be reused until purged. Acceptable.
4. **Custom domains.** Deleting a Static Web App or App Service releases the
   custom domain binding. DNS records may need manual cleanup.
5. **Multi-agent awareness.** If multiple agents (Claude, Codex, etc.) have
   created resources, do a full RG inventory before assuming nothing is orphaned.

## Red Flag: "I'll clean it up later"

This thought is the root cause of every orphaned resource. Cloud resources
left "for later" are never cleaned up. The cost is low (Free tier) but the
cognitive overhead of a cluttered resource group compounds. Delete now.
