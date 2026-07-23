---
name: azure-functions-deploy
description: "Deploy Azure Functions (TypeScript/Node) to Consumption plan via func CLI. Includes self-contained deploy package pattern, CORS config, and post-deploy verification."
version: 1.0.0
tags: [azure, functions, deployment, typescript, node]
platforms: [linux]
metadata:
  hermes:
    tags: [azure, functions, deployment, devops]
---

# Azure Functions Deploy (TypeScript/Node, Consumption plan)

## When to use
Deploying TypeScript Azure Functions v4 apps to a Consumption (Y1/Dynamic) plan.
Covers the full cycle: build, package, publish, configure CORS, verify.

## Verified patterns (from real deployments)

### Deploy method: func CLI only
```
func azure functionapp publish <APP_NAME> --node
```

NEVER use:
- `Azure/functions-action@v1` (uses Kudu zipdeploy internally → 503 "Function host is not running" on Consumption Linux)
- `az functionapp deployment source config-zip` (same Kudu path, same failure)
- `azd up` / `azd deploy` (requires azure.yaml infra scaffold — overkill for simple Functions apps)

### Self-contained deploy package (CRITICAL)
The pnpm workspace `workspace:*` dependency protocol cannot be resolved by `npm install`
on the Azure server. You must build a self-contained package:

```bash
# 1. Build the bundle locally
pnpm --filter @myapp/api build

# 2. Create deploy package OUTSIDE the workspace
rm -rf api/deploy-pkg && mkdir -p api/deploy-pkg
cp api/host.json api/deploy-pkg/
cp -r api/dist api/deploy-pkg/dist

# 3. Generate minimal package.json (no workspace:* deps)
node -e "
const p = require('./api/package.json');
require('fs').writeFileSync('api/deploy-pkg/package.json', JSON.stringify({
  name: p.name, version: p.version, main: 'dist/src/index.js',
  dependencies: { '@azure/functions': p.dependencies['@azure/functions'] }
}, null, 2));
"

# 4. Install production deps in the deploy package
cd api/deploy-pkg && npm install --omit=dev --no-audit --no-fund

# 5. Publish
func azure functionapp publish <APP_NAME> --node
```

### .funcignore (MUST NOT exclude dist)
```
.git
.vscode
local.settings.json
node_modules
# Do NOT add 'dist' here — the prebuilt bundle lives there
```

### Function registration guard
The Azure Functions v4 model only registers functions imported in `src/index.ts`.
A function file not imported compiles, tests green, and silently 404s in production.

Guard test (`src/index.test.ts`):
```typescript
it('imports every non-test module in src/functions', () => {
  const expected = readdirSync(functionsDir)
    .filter(f => f.endsWith('.ts') && !f.endsWith('.test.ts'))
    .map(f => f.replace(/\.ts$/, ''))
  const missing = expected.filter(name =>
    !new RegExp(`import\\s+['"]\\./functions/${name}['"]`).test(indexSource))
  expect(missing).toEqual([])
})
```

### WEBSITE_RUN_FROM_PACKAGE
If the function app returns 404 on ALL endpoints after deploy:
1. Check: `az functionapp config appsettings list -g <RG> -n <APP> --query "[?name=='WEBSITE_RUN_FROM_PACKAGE']"`
2. If set to a blob URL, clear it: `az functionapp config appsettings set -g <RG> -n <APP> --settings "WEBSITE_RUN_FROM_PACKAGE="`
3. Redeploy

### Trigger sync
If functions don't appear after deploy, force a sync:
```bash
az functionapp config appsettings set -g <RG> -n <APP> \
  --settings "AZURE_FUNCTIONS_SYNC_TRIGGERS=$((RANDOM % 100000))" --output none
sleep 15
```

## CORS configuration

### Consumption plan (Y1/Dynamic): platform CORS works
```bash
az functionapp cors add -g <RG> -n <APP> \
  --allowed-origins https://your-swa.azurestaticapps.net http://localhost:5173
```

### Flex Consumption: platform CORS is BROKEN
Kestrel front-end short-circuits OPTIONS preflights with empty 204 before function code runs.
Function-level cors.ts never executes for real browser preflights.
GitHub issues: azure-functions-host#5200, azure-functions-dotnet-worker#2524
Fix: use Consumption plan instead.

### Function-level CORS (application-level, in code)
```typescript
// cors.ts — runs on actual requests (not preflights on Flex)
const ALLOWED_ORIGINS = (process.env.ALLOWED_ORIGINS ?? 'http://localhost:5173')
  .split(',').map(o => o.trim())

export function withCors(response: HttpResponseInit, origin?: string): HttpResponseInit {
  const headers: Record<string, string> = {
    'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
    'Access-Control-Allow-Headers': 'Content-Type, x-sim-key',
    ...response.headers,
  }
  if (origin && ALLOWED_ORIGINS.includes(origin)) {
    headers['Access-Control-Allow-Origin'] = origin
  }
  return { ...response, headers }
}
```

## Post-deploy verification

```bash
# Health check (with retry for cold start ~60s)
for i in $(seq 1 6); do
  STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://<APP>.azurewebsites.net/api/health)
  [ "$STATUS" = "200" ] && echo "OK" && break
  echo "Attempt $i: $STATUS, waiting 15s..." && sleep 15
done

# List registered functions
func azure functionapp list-functions <APP_NAME>

# Count functions (should match expected)
az functionapp function list -g <RG> -n <APP> --query "length([])" -o tsv
```

## Azure Table Storage gotchas
- No array type — serialize arrays as JSON strings, deserialize on read
- Entity properties limited to primitive EDM types (string, number, boolean, datetime)
- RowKey/PartitionKey are strings — everything else needs explicit typing

## Monitoring setup
```bash
# App Insights alerts (one-liners)
az monitor metrics alert create -g <RG> -n api-error-rate-alert \
  --condition "avg HttpResultRate > 10" --scopes <APP_ID> --window-size 15m

az monitor metrics alert create -g <RG> -n api-response-time-alert \
  --condition "avg HttpLatency > 5000" --scopes <APP_ID> --window-size 15m
```
