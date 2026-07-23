---
name: azure-swa-deploy
description: "Deploy static sites to Azure Static Web Apps via GitHub Actions or SWA CLI. Covers framework detection, CI/CD, CORS with linked APIs, and production verification."
version: 1.0.0
tags: [azure, static-web-apps, swa, deployment, ci-cd]
platforms: [linux]
metadata:
  hermes:
    tags: [azure, swa, deployment, devops]
---

# Azure Static Web Apps Deploy

## When to use
Deploying frontend SPAs (vanilla TS, Vite, Next.js) to Azure Static Web Apps.
Covers GitHub Actions CI/CD, SWA CLI, linked API integration, and CORS.

## Deploy methods (by scenario)

### Method 1: GitHub Actions (recommended for CI/CD)
Use `Azure/static-web-apps-deploy@v1` action. Auto-detects framework from repo structure.

```yaml
# .github/workflows/deploy-frontend.yml
name: deploy-frontend
on:
  push:
    branches: [main]
    paths: ["frontend/**"]
  workflow_dispatch:

permissions:
  id-token: write
  contents: read

jobs:
  build-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v4
        with: { version: 9.12.0 }
      - uses: actions/setup-node@v4
        with: { node-version: "22", cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - name: Build
        run: pnpm --filter @myapp/frontend build
      - name: Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "frontend"
          output_location: "dist"
          skip_app_build: true
```

### Method 2: SWA CLI (for local dev / manual deploys)
```bash
# Install
npm install -g @azure/static-web-apps-cli

# Login
swa login --tenant-id <TENANT> --subscription-id <SUB>

# Deploy from build output
swa deploy ./frontend/dist --env production
```

## Framework detection (common configs)

| Framework | app_location | output_location | build command |
|-----------|-------------|-----------------|---------------|
| Vite (vanilla TS) | `frontend` | `dist` | `vite build` |
| Vite (React) | `src` | `dist` | `npm run build` |
| Next.js | `/` | `.next` | `npm run build` |
| Astro | `/` | `dist` | `npm run build` |

## Linked API integration (SWA + Azure Functions)

### Free tier: NO managed API proxy
SWA Free does NOT support linking an external Function App as a managed backend.
Calls to `/api/*` will not be proxied — the browser makes cross-origin requests
to the Function App directly, requiring CORS configuration.

### Standard tier: managed API proxy available
```bash
az staticwebapp appsetting set --name <SWA_NAME> \
  --setting-names "AZURE_FUNCTIONS_API_BASE_URL=https://my-func.azurewebsites.net"
```
With managed proxy, browser calls go to same-origin `/api/*` — no CORS preflight needed.

### Cross-origin calls (Free tier pattern)
When the frontend and API are on different origins, configure CORS on the Function App:
```bash
az functionapp cors add -g <RG> -n <FUNC_APP> \
  --allowed-origins https://<SWA_NAME>.azurestaticapps.net
```

## Environment variables

### Build-time (Vite)
```bash
# .env or GitHub Actions env
VITE_API_BASE_URL=https://my-func.azurewebsites.net/api
VITE_MOCK=0
```

### Runtime (App Settings)
```bash
az staticwebapp appsetting set --name <SWA_NAME> \
  --setting-names "VITE_API_BASE_URL=https://my-func.azurewebsites.net/api"
```

## CI/CD contract

### Pre-deploy checks
1. `pnpm install --frozen-lockfile` succeeds
2. `pnpm -r typecheck` passes
3. `pnpm -r test` passes
4. Build output exists (`frontend/dist/index.html`)

### Post-deploy verification
```bash
SWA_URL="https://<SWA_NAME>.azurestaticapps.net"

# Check frontend loads
STATUS=$(curl -s -o /dev/null -w "%{http_code}" "$SWA_URL")
[ "$STATUS" = "200" ] || echo "FAIL: SWA returned $STATUS"

# Check JS bundle deployed
BUNDLE=$(curl -s "$SWA_URL" | grep -oP 'src="\K[^"]+\.js' | head -1)
curl -s -o /dev/null -w "%{http_code}" "$SWA_URL$BUNDLE"

# Check nav links in production HTML
curl -s "$SWA_URL" | grep -oP 'data-route="\w+"' | sort
```

## Custom domains

```bash
# Add custom domain
az staticwebapp custom-domain create \
  --name <SWA_NAME> --resource-group <RG> \
  --hostname myapp.example.com

# If using a linked API, also configure CORS for the custom domain
az functionapp cors add -g <RG> -n <FUNC_APP> \
  --allowed-origins https://myapp.example.com
```

## Common issues

### "NetworkError when attempting to fetch resource"
Cross-origin API call blocked by CORS. Fix: add the SWA origin to Function App CORS.

### Build succeeds but page is blank
- Check `output_location` matches your build output directory
- For Vite: `dist`, not `build`
- Verify the bundle is referenced in the deployed HTML

### SWA CLI deploy times out
Large bundles (>50MB) can time out. Split into chunks or use GitHub Actions instead.

### App not updating after deploy
SWA CDN caches aggressively. Hard-refresh, or append a cache-buster to verify.
