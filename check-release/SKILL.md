---
name: check-release
description: Pre-release checklist before deploying to production. Covers env vars, secrets, storage schema changes, dependency updates, rollback plan, and post-deploy verification. Run this before every production deploy. Triggers on: "pre-release check", "ready to deploy", "release checklist", "deploy checklist".
---

# check-release: Pre-Release Checklist

Run a pre-release check for `$ARGUMENTS` (version, branch, or feature description) before deploying to production.

**Step 1** — Review git log and changed files since the last production deploy.

## Storage & Data

- [ ] Any Azure Table Storage schema changes are additive-only (no renamed/removed properties that break readers)
- [ ] Seed data or fixture updates committed and tested against staging
- [ ] No hardcoded partition/row key values that will conflict with existing production data
- [ ] Storage account access keys / connection strings not rotated without updating app settings first

## Configuration & Secrets

- [ ] All new environment variables added to Azure App Settings / Function App config
- [ ] No new `process.env.*` references without a corresponding secret/var in the deploy target
- [ ] `.env.example` updated if new vars were added
- [ ] Key Vault references resolve correctly in the target environment
- [ ] `ANTHROPIC_API_KEY` / third-party API keys present and valid if the feature uses AI

## Dependencies

- [ ] `npm audit` shows no critical/high vulnerabilities in production dependencies
- [ ] Lockfile (`package-lock.json`) committed and matches `package.json`
- [ ] No `*` or `latest` version pins that could cause non-deterministic builds

## Code & Build

- [ ] TypeScript builds without errors (`tsc --noEmit`)
- [ ] All tests pass (unit + e2e / smoke)
- [ ] No `console.log` debug output left in hot paths
- [ ] No commented-out code or TODO blocks introduced in this release

## Azure Deployment

- [ ] GitHub Actions workflow `workflow_dispatch` trigger works (manual re-run tested)
- [ ] SWA deploy: `app_location` and `api_location` correct for this release
- [ ] Functions deploy: `WEBSITE_RUN_FROM_PACKAGE` set, package size is reasonable
- [ ] CORS platform config includes the production origin (`az functionapp cors show`)
- [ ] SCM basic auth enabled if using Kudu/zip deploy
- [ ] Flex Consumption: cold-start behaviour acceptable for this feature's latency requirements

## Rollback Plan

- [ ] Previous working commit/tag identified and noted
- [ ] Rollback steps documented (redeploy prior package / revert app settings)
- [ ] Stateful changes (storage schema, data) are reversible or acceptable to leave

## Post-Deploy Verification

- [ ] Health/smoke endpoint returns 200 within 60s of deploy (account for cold start)
- [ ] Key user flows manually tested in production (golden path)
- [ ] No spike in errors visible in Application Insights / Azure Monitor after 5 minutes
- [ ] Feature flags / environment toggles in correct state for production

---

**Do not deploy until all Critical items are checked. Minor items can be tracked as follow-up.**
