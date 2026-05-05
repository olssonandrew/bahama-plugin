# Vite + Hono

Use `vite-hono` for full-stack Bahama apps: a Vite frontend served as static assets plus a Workers-compatible Hono backend for `/api/*` routes.

Read this file when choosing or implementing the `vite-hono` deployment type. For detailed D1, SQL, secrets, or local testing guidance, read `database-and-sql.md`, `secrets.md`, or `local-development.md`.

## When To Use

Use `vite-hono` when the app needs any of these:

- persistent data through Bahama-managed D1
- server-side API keys or third-party provider secrets
- AI provider calls that must not expose keys to browser code
- webhooks or JSON API routes
- CRUD behavior where the frontend calls backend routes
- a Vite frontend and a small server-side surface in the same project

Do not use `vite-hono` for browser-only apps. Use `vite-spa` when there is no database, no server-side secret, no webhook, and no backend route.

## Required Shape

The project must include:

- `package.json`
- one lockfile: `package-lock.json`, `pnpm-lock.yaml`, or `yarn.lock`
- `index.html`
- `src/`
- Vite dependency
- Hono dependency
- build script
- deployable backend entry at `server/index.ts`, `server/index.js`, `server/index.mts`, or `server/index.tsx`

The frontend must build with Vite. For React projects, use Vite. Do not use Next.js, Remix, custom Webpack, or SSR framework adapters unless Bahama explicitly supports them.

## Vite Rules

Use Vite as the frontend build system.

- Keep the app entry in `src/`.
- Keep the HTML entry at project-root `index.html`.
- Use normal Vite scripts, usually `dev`, `build`, and optionally `preview`.
- Keep frontend environment variables public-only; Vite exposes `VITE_*` variables to browser code.
- Do not put private provider keys, Bahama dev tokens, D1 details, or deployment credentials in `VITE_*` variables.
- Let Bahama build the app. Do not package prebuilt output for this type.

For React, create or migrate to a Vite React app. This is the supported React path.

## Project State

Before coding or deploying, ensure that the Bahama project metadata matches the architecture with MCP `bahama_get_project` and `bahama_update_project`

- Set backend to `hono` for `vite-hono`.
- Enable D1 only when the app needs persistent data.
- Provision D1 before writing or deploying code that expects `env.DB`.
- Package and deploy according to this `vite-hono` contract.

Do not ask the user for credentials, database URLs, hosts, passwords, or connection strings. Bahama binds managed resources into the deployed Worker.

## Routing Model

Bahama serves the Vite build output as static assets and routes `/api/*` through the Hono backend.

- Put browser UI in the Vite frontend.
- Put server behavior in Hono routes under `/api/*`.
- Have the frontend call backend routes with relative paths like `/api/notes`, `/api/message`, or `/api/generate`.
- Do not configure the Hono backend to serve the frontend assets.

## Frontend API Rules

Keep browser code as browser code.

- Call backend routes with relative paths like `/api/notes`, `/api/message`, or `/api/generate`.
- Do not import server-only SDKs from frontend files.
- Do not read `env.DB`, `env.SECRET_NAME`, `BAHAMA_DEV_TOKEN`, or any private token from browser code.
- Do not put private keys in `VITE_*` variables.
- Handle API errors explicitly in the UI; do not assume every response is JSON success.

Good frontend call pattern:

```ts
const response = await fetch("/api/notes");
const payload = await response.json();

if (!response.ok) {
  throw new Error(payload.error ?? "Unable to load notes.");
}
```

## Hono Rules

The deployable backend entry must be Workers-compatible.

Allowed in `server/index.*`:

- `import {Hono} from "hono"`
- `export default app`
- route handlers under `/api/*`
- access to D1 through `c.env.DB` or `getDb(c.env)`
- access to project secrets through `c.env.SECRET_NAME`

Not allowed in `server/index.*`:

- `@hono/node-server`
- `serve(...)`
- `serveStatic(...)`
- Express
- Node HTTP server setup
- filesystem-dependent production routing
- long-running Node server assumptions

Use a separate local adapter such as `server/dev.ts` for Node local development. Keep Node adapters out of the deployable entry.

Minimal deployable Hono entry:

```ts
import {Hono} from "hono";

type Env = {
  Bindings: {
    DB?: D1Database;
    OPENAI_API_KEY?: string;
  };
};

const app = new Hono<Env>();

app.get("/api/health", (c) => {
  return c.json({ok: true});
});

app.get("/api/message", (c) => {
  return c.json({message: "Hello from the Hono backend."});
});

export default app;
```

## Conditional Details

- If adding SQL, persistent CRUD, migrations, seed data, or D1 access, read the database/SQL reference before writing code.
- If adding third-party API keys or server-side credentials, read the secrets reference before writing code.
- If setting up local Hono development against Bahama-managed resources, read the local testing reference before creating dev tokens or `.env.local`.

## Packaging

Package source, not local build artifacts.

Include:

- `package.json`
- one lockfile
- `index.html`
- `src/`
- `server/index.*`
- `public/` when used
- Vite config and TypeScript config when used

Exclude:

- `node_modules/`
- `dist/`
- `.git/`
- `.env*`
- `bahama.json`
- `coverage/`
- `*.log`
- `__MACOSX/`
- `.DS_Store`
- local screenshots, recordings, experiments, or other files that are not part of the app

Zip the project root contents, not the parent directory. Request an upload URL, upload the zip, start the deploy, and poll deploy status until terminal when possible.
