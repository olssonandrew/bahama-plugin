---
name: bahama-builder
description: Build, provision, test locally, and deploy Bahama projects through the Bahama MCP server. Use when creating or updating apps that should run on Bahama, provisioning project storage, preparing a project for deploy, or deploying a React + Vite app with an optional Workers-compatible Hono backend. Use when the user wants to go from idea to running Bahama app, especially for notes apps, CRUD apps, frontend apps with APIs, or any workflow that needs a Bahama MCP connection, project slug, storage provisioning, source packaging, upload, deploy, and deployment status polling.
---

# Bahama Builder

Bahama is an agentic infrastructure platform. It allows an agent to create or load a project, manage its resources, prepare source code for deployment, and deploy a working application with very little human effort. Use the Bahama MCP server as the system of action. Do not call infrastructure provider APIs directly for normal Bahama workflows.

## Start here

Before doing anything else, make sure you have:

- the Bahama MCP plugin installed and available
- a working OAuth connection to Bahama
- a Bahama project slug when updating an existing project

If the Bahama MCP tools are not available or the user is not connected:

- tell them they need to connect Bahama and complete the OAuth flow before Bahama operations can work
- do not invent or bypass auth

If the user does not know their project slug:

- ask whether they already have a Bahama project
- if they do, use the MCP to look it up or confirm it
- if they do not, create a new project using their intended app concept and a sensible slug

Do not guess a random project. Use a stable slug.

## Use the MCP, not ad hoc requests

Use the Bahama MCP server as the system of action.

The Bahama MCP should expose these tools:

- `bahama_create_project`
- `bahama_get_project`
- `bahama_update_project`
- `bahama_get_runtime`
- `bahama_get_database`
- `bahama_provision_database`
- `bahama_query_database`
- `bahama_create_upload_url`
- `bahama_start_deploy`
- `bahama_get_deploy_status`
- `bahama_get_deploy_instructions`
- `bahama_create_dev_token`


If these tools are missing, stop and explain that the Bahama MCP connection is incomplete.

## Supported app contract

Bahama currently supports:

- React + Vite frontend apps
- optional Hono backend apps under a strict format

Do not build for any other framework unless the user explicitly says Bahama has been extended to support it.

## React + Vite requirements

The app must include:

- `package.json`
- one lockfile
- `src/`
- `index.html`

It may include:

- `public/`
- `vite.config.*`
- `tsconfig.json`
- `server/` when using Hono

## Hono rules

Bahama supports user-authored Hono backends only in a specific format.

Allowed:

- backend entry at `server/index.ts`, `server/index.js`, `server/index.mts`, or `server/index.tsx`
- Workers-compatible Hono app
- `export default app`
- Hono route handlers that use `env.DB` when DB is provisioned

Not allowed in the deployable backend entry:

- `@hono/node-server`
- `serve(...)`
- `serveStatic(...)`
- Express
- custom Node HTTP server setup
- any production backend code that expects a long-running Node server process

Local development is allowed to use a separate adapter file such as `server/dev.ts`. The deployable entry is still `server/index.*`.

Important:

- local server-side database access is available through `@bahama-ai/sdk`
- local SDK access uses the project’s live Bahama-managed D1 database through Bahama’s dev proxy
- deployed Worker code still uses native `env.DB`
- never expose `BAHAMA_DEV_TOKEN` in browser code, `VITE_*` variables, committed files, or deploy bundles

Good deployable pattern:

```ts
import {Hono} from "hono";

type Env = {
  Bindings: {
    DB: D1Database;
  };
};

const app = new Hono<Env>();

app.get("/api/message", (c) => {
  return c.json({message: "Hello from the Hono backend!"});
});

export default app;
```

## Database Storage Rules

If the app needs a database:

- mark the project as needing DB storage through the Bahama project update flow
- provision DB storage through the Bahama MCP
- then write backend code that uses `env.DB`

Do not ask the user for:

- a database URL
- a host
- a password
- a connection string

Bahama binds DB into the runtime when the project state says it exists.

The frontend must never talk to the database directly. Use backend routes.

Minimal useful Hono + db storage pattern:

```ts
import {Hono} from "hono";

type Env = {
  Bindings: {
    DB: D1Database;
  };
};

const app = new Hono<Env>();

app.get("/api/notes", async (c) => {
  const {results} = await c.env.DB.prepare(
    "SELECT id, text, created_at FROM notes ORDER BY id DESC",
  ).all();

  return c.json({
    notes: results ?? [],
  });
});

app.post("/api/notes", async (c) => {
  const body = await c.req.json();
  const text = typeof body?.text === "string" ? body.text.trim() : "";

  if (!text) {
    return c.json({error: "text is required"}, 400);
  }

  await c.env.DB.prepare("INSERT INTO notes (text, created_at) VALUES (?, ?)")
    .bind(text, new Date().toISOString())
    .run();

  return c.json({ok: true});
});

export default app;
```

If the app uses a DB, keep all reads and writes in Hono route handlers. Have the frontend call relative API routes like `/api/notes`.


## Local development with live resources

Bahama DBs use bindings that are not easily compatible with local testing. To enable testing, we have created a development SDK that proxies live database access on local testing. Note that this is LIVE project data, so be cautious to avoid destructive operations or test data pollution. 

1. Provision D1 with `bahama_provision_database`.
2. Create a dev token with `bahama_create_dev_token`.
3. Write the returned values to `.env.local`:
   - `BAHAMA_API_BASE_URL`
   - `BAHAMA_PROJECT_SLUG`
   - `BAHAMA_DEV_TOKEN`
4. Install the SDK:
   - `npm install @bahama-ai/sdk`
5. Use `@bahama-ai/sdk/server` from server-side code only.

Use this pattern in Hono/server code:

```ts
import {Hono} from "hono";
import {getDb} from "@bahama-ai/sdk/server";

type Env = {
  Bindings: {
    DB?: D1Database;
    BAHAMA_API_BASE_URL?: string;
    BAHAMA_PROJECT_SLUG?: string;
    BAHAMA_DEV_TOKEN?: string;
  };
};

const app = new Hono<Env>();

app.get("/api/notes", async (c) => {
  const db = getDb(c.env);
  const {results} = await db
    .prepare("SELECT id, text, created_at FROM notes ORDER BY id DESC")
    .all();

  return c.json({notes: results ?? []});
});

export default app;

```

## Project lifecycle

Use this order.

### 1. Load or create the project

Get the current project if the user already has one.

If not, create a new project with:

- a good slug
- `framework = react-vite`
- `backend = none` for frontend-only apps or `backend = hono` for apps needing server routes
- `d1_enabled = true` only when DB is actually needed

### 2. Update project metadata before coding or deploy

Do not wait until deploy to decide architecture if you can avoid it.

If the app needs Hono or DB, update the project first so Bahama is the source of truth.

Typical update:

- set `backend` to `hono`
- set `d1_enabled = true` if the app needs persistent data

### 3. Provision resources only when needed

If the app needs persistence:

- call the Bahama DB storage provisioning tool
- optionally run a smoke-test SQL query afterward

If the app is frontend-only:

- do not provision DB storage

### 4. Build the app to the Bahama contract

For frontend-only apps:

- write a normal React + Vite app
- use relative API paths only if the app is meant to call backend routes later

For Hono apps:

- write the React + Vite frontend
- put deployable backend logic in `server/index.*`
- have the frontend call relative API routes like `/api/message` or `/api/notes`

### 5. Package and deploy

- request deploy instructions when you need the current packaging contract
- request an upload URL through the Bahama MCP
- zip the project root contents cleanly
- upload the zip to the returned upload target
- start the deploy
- poll deploy status until terminal when possible

When preparing the zip:

- zip the project root contents, not the parent directory
- include only source code, build config, and files needed for install/build
- do not prebuild before upload
- do not include generated output or machine-local clutter

Specifically exclude:

- `node_modules/`
- `dist/`
- `.git/`
- `__MACOSX/`
- `.DS_Store`
- `coverage/`
- `*.log`
- `.env*`
- editor temp files
- OS metadata files
- screenshots, recordings, exports, or notes that are not part of the build
- large local assets that are not referenced by the app
- any extra folders that exist only for local experimentation or debugging

If there is any doubt, include only the files needed to run `install` and `build`.

In practice, Bahama deploys are usually fast. Many complete in roughly 15-30 seconds. Poll for status when you can, but do not treat a missed poll as fatal; other reconciliation mechanisms may still resolve the final state later.

## Behavior guidelines

- Prefer using existing Bahama project state over guessing.
- Use the Bahama MCP tools for provisioning and deploy operations.
- If required auth or project context is missing, stop and ask the user for it.
- If the user does not have a project yet, create one with their app idea in mind.
- Keep the app within the supported Bahama framework contract.
- Do not mention or require provider-specific deployment config in the app project.
- Do not ask the user to manually wire deployment bindings.
- Do not rely on unsupported backend formats.
