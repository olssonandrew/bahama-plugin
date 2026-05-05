# Database And SQL

Use this file before adding D1, SQL tables, migrations, seed data, or persistent CRUD behavior.

Bahama-managed D1 is a runtime binding for server-side Worker/Hono code. Browser code must never talk to D1 directly.

## Setup Order

1. Decide whether persistence is actually needed.
2. Update the Bahama project so D1 is enabled.
3. Provision the database through Bahama MCP.
4. Write server-side Hono routes that use the database binding.
5. Run schema setup or migrations deliberately.
6. Have frontend code call relative API routes such as `/api/notes`.

Do not ask the user for a database URL, host, password, username, or connection string. Bahama owns provisioning and runtime binding.

## Runtime Access

In deployed Worker/Hono code, D1 is usually available as `env.DB`.

For code that also needs local testing through Bahama's dev proxy, prefer `@bahama-ai/sdk/server` and `getDb(c.env)`. See `/references/local-development.md` for details.

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

If local testing is not needed, direct `c.env.DB` access is acceptable in deployable Hono code.

## Schema Setup

Create tables deliberately before relying on them in app routes. Prefer running setup or migration SQL through Bahama MCP so schema changes are explicit and visible.

For small prototypes, it is acceptable to add an idempotent setup helper that uses `CREATE TABLE IF NOT EXISTS` before reads or writes. Keep that helper narrow, safe to run repeatedly, and limited to the tables the route actually needs.

```ts
async function ensureNotesTable(db: D1Database) {
  await db
    .prepare(
      `CREATE TABLE IF NOT EXISTS notes (
        id INTEGER PRIMARY KEY AUTOINCREMENT,
        text TEXT NOT NULL,
        created_at TEXT NOT NULL
      )`,
    )
    .run();
}
```

Use parameter binding for user input. Do not concatenate user-provided values into SQL strings.

```ts
await db
  .prepare("INSERT INTO notes (text, created_at) VALUES (?, ?)")
  .bind(text, new Date().toISOString())
  .run();
```

## Frontend Boundary

Frontend code should:

- call backend routes
- display returned data
- handle loading and error states

Frontend code must not:

- import database helpers
- read `env.DB`
- use Bahama dev tokens
- receive raw database credentials

## Safe Querying

Use `bahama_query_database` for setup checks, schema creation, seed data, or debugging when the user has authorized the operation.

Be careful with destructive SQL. Ask before dropping tables, deleting broad data, or overwriting user records.
