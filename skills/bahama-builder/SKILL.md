---
name: bahama-builder
description: Build, provision, test, package, and deploy web services through the agent-native Bahama cloud platform. Use when creating, updating, or managing web apps that should run on Bahama.
---

# Bahama Builder

Bahama is an agentic infrastructure platform. It allows an AI agent to create or load a project, manage its resources, prepare source code for deployment, and deploy a working application with minimal human effort. Use the Bahama MCP server as the system of action. Do not call infrastructure provider APIs directly for normal Bahama workflows.

## Project Selection And Setup

Every Bahama app must be linked to exactly one Bahama project before coding, provisioning, local testing, or deploying.

Use a repo-root `bahama.json` file as the local project binding.

```json
{
  "projectSlug": "slug-name",
  "deploymentType": "deployment-type"
}
```

Use only `projectSlug` and `deploymentType` in this file. Do not store dev tokens, secrets, database IDs, upload IDs, or deploy job IDs in `bahama.json`.

Before changing code or resources:

1. Look for `bahama.json` in the repo root.
2. If it exists, treat `projectSlug` as the intended Bahama project for this folder and confirm it with the MCP `bahama_get_project` to confirm it exists and uses the correct deployment type before mutating resources or deploying.
3. If this file does not exist, confirm with the user whether they already have a Bahama project slug for this app, or whether they need to create a new one.
4. If an existing project exists, get the slug, choose or confirm the deployment type, confirm the project through MCP `bahama_get_project`, and create `bahama.json`.
5. If no Bahama project exists, confirm a new name with the user (lowercase slug with only letters, numbers, and dashes). Choose the deployment type, create the Bahama project via the MCP tool `bahama_create_project` with all required parameters, then create `bahama.json` once successful.
6. Begin building the app according to the selected deployment type, the rules of this skill, and the user's requirements.

Never deploy, provision D1, create dev tokens, query SQL, or direct the user to add project secrets until the slug has been resolved and confirmed.

## Bahama MCP Server

Use the Bahama MCP server as the system of action. It should be installed and authenticated before going further.

The MCP tools enable you to act on the user's Bahama account to get, create, and update projects; provision and manage D1 databases; enable local testing of remote resources; and manage the deployment process that puts code live on `https://<slug>.bahama.app` or a custom domain.

Always use the MCP server to interact with the user's Bahama projects. If the MCP tools are missing, stop and explain that the Bahama MCP connection is incomplete. Do not invent auth, bypass OAuth, or ask for credentials directly.

## Deployment Types

Choose one supported deployment type before coding.

- `vite-hono`: Vite frontend plus Workers-compatible Hono backend on `/api/*`. Default for full-stack apps, CRUD apps, D1-backed apps, webhooks, AI/provider calls, and apps needing server-side secrets. Read `references/vite-hono.md`.
- `vite-spa`: Vite frontend only. Use for browser-only React, Vue, Svelte, Preact, Solid, or vanilla Vite apps with no DB, no server-side secrets, and no backend routes. Read `references/static-deployments.md`.
- `static-site`: Raw HTML/CSS/JS with no package install and no build step. Use for simple browser-only sites. Read `references/static-deployments.md`.
- `static-bundle`: Already-built static assets with `index.html` at root, `dist/`, `build/`, or `public/`. Use when another tool already produced deployable output. Read `references/static-deployments.md`.
- `hono-api`: Backend-only Hono Worker API with no frontend assets. Use for JSON APIs, webhooks, automation endpoints, and service backends. Read `references/hono-api.md`.

For React, use Vite. Do not use Next.js, Remix, Nuxt, custom Webpack, Express, Node HTTP servers, or SSR framework adapters unless Bahama explicitly supports them. For existing unsupported projects, assess whether they can be cleanly converted to a supported type and discuss the conversion before rewriting.

## Data Rule

Bahama-managed D1 is available only to server-side Worker/Hono code. Browser code must call backend routes. Never ask the user for a database URL, host, password, or connection string.

If adding SQL, D1 tables, migrations, seed data, or persistent CRUD behavior, read `references/database-and-sql.md` before writing code.

## Secrets Rule

Bahama project secrets are write-only runtime values for third-party credentials. Do not ask the user to paste raw secret values into chat. Instead, choose the exact secret name, tell the user to add it at `/dashboard/projects/:slug/secrets`, and read it only from server-side Worker/Hono code as `env.SECRET_NAME`.

If adding provider keys, OAuth client secrets, webhook signing secrets, or local testing with secrets, read `references/secrets.md`.

## Local Testing Rule

Bahama local testing can use live Bahama-managed resources through dev tokens and `@bahama-ai/sdk/server`. Dev tokens and secret values are server-side local configuration only.

If setting up local Hono development, local D1 access, Vite API proxying, or `.env.local`, read `references/local-development.md`.

## Deployment Workflow

Use this order:

1. Confirm MCP tools and auth.
2. Load or create the Bahama project.
3. Choose the deployment type and read the matching reference file.
4. Update project metadata before coding when backend or D1 needs are known.
5. Provision D1 only if the app needs persistence.
6. Add secrets through the dashboard path when server-side credentials are needed.
7. Build or adjust the app to the selected Bahama contract.
8. Package the archive according to the deployment type.
9. Create an upload URL, upload the zip, start deploy, and poll status until terminal when possible.

Read `references/packaging-and-deploy.md` before zipping, uploading, starting deploy, or troubleshooting deploy failures.

## General Build Rules

- Keep generated apps within the selected Bahama deployment type.
- Keep frontend code separate from server-only resource access.
- Use relative frontend API paths like `/api/notes` for Bahama backend routes.
- Do not expose D1, dev tokens, project secrets, provider keys, or credentials to browser code.
- Do not add provider-specific deployment config to the app unless Bahama explicitly supports it.
- Do not rely on unsupported backend formats or long-running Node server processes.
- Prefer this skill's Bahama contract and the selected reference file over stale local assumptions.

## Reference Files

- `references/vite-hono.md`: Read for Vite frontend plus Hono backend apps.
- `references/static-deployments.md`: Read for `static-site`, `static-bundle`, and `vite-spa`.
- `references/hono-api.md`: Read for backend-only Hono API deployments.
- `references/database-and-sql.md`: Read before adding D1, SQL, migrations, seed data, or persistent CRUD.
- `references/secrets.md`: Read before using server-side provider credentials or local secret values.
- `references/local-development.md`: Read before using dev tokens, `@bahama-ai/sdk`, `.env.local`, or local Hono/Vite API proxying.
- `references/packaging-and-deploy.md`: Read before creating deploy archives or troubleshooting deploy jobs.
