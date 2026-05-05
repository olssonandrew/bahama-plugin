# Packaging And Deploy

Use this file before creating a deploy archive, requesting an upload URL, starting a deploy, polling status, or troubleshooting deployment failures.

## Deploy Flow

1. Confirm the project slug.
2. Confirm the deployment type.
3. Zip the correct files according to this reference and the selected deployment type.
4. Use the Bahama MCP tool `bahama_create_upload_url` to create an upload target.
5. Upload the zip to the returned upload target.
6. Use the Bahama MCP tool `bahama_start_deploy` with the returned upload ID.
7. Use the Bahama MCP tool `bahama_get_deploy_status` to poll until `deployed` or `failed` when possible.

Do not pass large source archives through MCP tool bodies. Use the upload URL flow.

## Zip Rules

Zip the project root contents, not the parent directory.

Good:

```text
index.html
package.json
src/
server/
```

Bad:

```text
my-project/index.html
my-project/package.json
```

Bahama strips a single wrapper directory in some cases, but do not rely on that when creating archives.

## Always Exclude

- `node_modules/`
- `.git/`
- `.env*`
- `coverage/`
- logs
- `.DS_Store`
- `__MACOSX/`
- editor temp files
- screenshots, recordings, exports, notes, or local experiments that are not part of the app
- large unreferenced local assets

Never include dev tokens or raw secrets.

## Type-Specific Packaging

`static-site`:

- include root `index.html` and referenced assets
- no install or build step
- no `server/index.*`

`static-bundle`:

- include already-built deployable output
- `index.html` must be at root, `dist/`, `build/`, or `public/`
- no install or build step
- no `server/index.*`

`vite-spa`:

- include source and build config
- include `package.json`, one lockfile, root `index.html`, `src/`, and Vite config when used
- exclude `dist/`
- Bahama builds to `dist/index.html`

`vite-hono`:

- include Vite frontend source and `server/index.*`
- include `package.json`, one lockfile, root `index.html`, `src/`, and server modules
- exclude `dist/`
- Bahama builds frontend assets and bundles Hono

`hono-api`:

- include `package.json`, one lockfile, `server/index.*`, and server modules
- no static frontend assets required
- Bahama bundles Hono

## Status And Troubleshooting

Poll deploy status until terminal when possible. If polling misses the final update, do not assume deploy failed; read the latest status again later.

Common failure meanings:

- `invalid_upload`: archive shape is wrong, missing required files, unsafe paths, or wrong deployment type
- `unsupported_framework`: unsupported dependency or backend shape, such as `server/index.*` in a static deployment
- `unsupported_package_manager`: missing supported lockfile
- `dependency_install_failed`: package install failed in the sandbox
- `build_failed`: Vite build or Hono bundle failed
- `deploy_failed`: publish failed inside Bahama deployer
- `smoke_test_failed`: deploy published but live route did not pass readiness checks

When a deploy fails, inspect the job logs and fix the source contract first. Do not switch frameworks casually to bypass a contract error.
