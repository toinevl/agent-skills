---
name: tvv-ops-pipeline
description: 'Review GitHub Actions CI/CD pipeline configuration for correctness, security, caching, and efficiency. Use before merging workflow changes or when a pipeline is slow, flaky, or leaking secrets. Triggers on: "review workflow", "check pipeline", "GitHub Actions review", "CI review".'
version: 1.0.0
category: ops
scope: [all]
triggers: [review workflow, check pipeline, GitHub Actions review, CI review]
status: stable
created: '2026-07-23'
last_updated: '2026-07-29'
---

# tvv-ops-pipeline: CI/CD Pipeline Review

Review the pipeline in `$ARGUMENTS` (workflow file path or description).

**Step 1** — Read the workflow file(s) and identify the platform (GitHub Actions assumed) and what the pipeline does.

## Structure & Stages

- Pipeline has clear, distinct jobs: build → test → deploy
- Deploy jobs depend explicitly on test jobs passing (`needs:`)
- Production deploy requires manual approval or environment protection rules
- Branch filters are intentional — `main` deploys to prod, PRs do not

## Security

- No secrets hardcoded in YAML — all sensitive values via `${{ secrets.* }}`
- `permissions:` scoped to minimum required (avoid `write-all`)
- Third-party actions pinned to a full commit SHA, not a mutable tag
- `pull_request_target` not used without careful review (RCE risk)
- Environment secrets scoped to the correct environment (not available to PR builds)

## Azure Deployments

- SWA deploy uses `azure/static-web-apps-deploy@v1` with correct `app_location` / `api_location`
- Azure Functions deploy: package built and uploaded correctly; `SCM_DO_BUILD_DURING_DEPLOYMENT` matches intent
- `AZURE_CREDENTIALS` / publish profile stored as a secret, not in env vars
- Post-deploy smoke test or health check step present before marking deploy successful
- Deployment slots or blue/green pattern used for zero-downtime where needed

## Efficiency

- `node_modules` / build output cached with a cache key that includes lockfile hash
- Jobs parallelized where independent (typecheck, unit tests, e2e can run in parallel)
- No redundant `npm install` / checkout steps across jobs that could share artifacts
- Jobs that always run (even on failure) use `if: always()` explicitly

## Reliability

- `workflow_dispatch:` trigger present so pipelines can be run manually
- Timeout set on long-running jobs (`timeout-minutes:`)
- Flaky steps (external service calls, deploys) have retry logic or `continue-on-error` documented

---

**Output findings as: Critical (blocks merge) > Major (fix soon) > Minor (nice to have).** Include `file:line`.
