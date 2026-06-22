# Managed Apps GitHub Actions

GitHub Actions for building and deploying [Microsoft Managed Apps](https://www.npmjs.com/package/@microsoft/managed-apps-cli) from CI.

## Actions

| Action | What it does | Ref |
|---|---|---|
| [`install-ms-cli`](./install-ms-cli) | Installs `@microsoft/managed-apps-cli` (binary: `ms`) on the runner. | `microsoft/Managed-Apps/github-actions/install-ms-cli@v1` |
| [`ms-app-pack`](./ms-app-pack) | Runs `ms app pack` — builds and packs the app into a deployable artifact. | `microsoft/Managed-Apps/github-actions/ms-app-pack@v1` |
| [`ms-app-deploy`](./ms-app-deploy) | Runs `ms app deploy` — uploads the artifact and deploys to the target environment. | `microsoft/Managed-Apps/github-actions/ms-app-deploy@v1` |

## Quick start

```yaml
name: Deploy Managed App

on:
  push:
    branches: [main]
    paths:
      - 'apps/my-app/**'
      - '.github/workflows/deploy-managed-app.yml'
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - uses: actions/setup-node@v4
      with: { node-version: '22' }
    - run: npm install
      working-directory: apps/my-app
    - uses: microsoft/Managed-Apps/github-actions/install-ms-cli@v1
    - uses: microsoft/Managed-Apps/github-actions/ms-app-pack@v1
      with:
        working-directory: 'apps/my-app'
        app-id:        ${{ secrets.PP_SP_CLIENT_ID }}
        client-secret: ${{ secrets.PP_SP_CLIENT_SECRET }}
        tenant-id:     ${{ secrets.PP_SP_TENANT_ID }}
    - uses: microsoft/Managed-Apps/github-actions/ms-app-deploy@v1
      with:
        working-directory: 'apps/my-app'
        app-id:        ${{ secrets.PP_SP_CLIENT_ID }}
        client-secret: ${{ secrets.PP_SP_CLIENT_SECRET }}
        tenant-id:     ${{ secrets.PP_SP_TENANT_ID }}
```

See each action's `action.yml` for the full input/output reference.

## Architecture

See [Architecture & Flow](.claude/skills/architecture.md) for the end-to-end MAAF lifecycle (one-time setup, local dev, and CI loops), the `ms-app-deploy` sequence with RP rejection points, and the release/versioning flow — all with diagrams.

## Prerequisites

- A **Service Principal** in Microsoft Entra ID with the right role on your target Power Platform environment (System Administrator + System Customizer for Dataverse-enabled environments; Environment Admin for non-Dataverse environments via the BAP REST API).
- **`AllowExternalArtifactDeployment`** enabled on the target environment by your tenant administrator.
- A Managed App created with `ms app create --repo none` (BYOB / escape-hatch mode), `ms.config.json` committed in the repo.

## Building the actions

```pwsh
cd github-actions
npm install
npm run build
npm run dist
```

This compiles the TypeScript sources under `src/actions/` and bundles each action into a single JS file under `dist/actions/<action-name>/index.js` via esbuild.

Each action's `action.yml` references the bundled `dist/actions/<name>/index.js` as its entry point.

## Versioning

Actions follow the repository's release tags. Consumers can pin to:

- `@v1` — major version, gets new features + non-breaking changes
- `@v1.x.y` — specific immutable version, never moves

## License

[MIT](../LICENSE)
