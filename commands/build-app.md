---
description: Use Bahama MCP plus the Bahama Builder skill to create, provision, and deploy a Bahama app.
---

Use the `bahama-builder` skill and the Bahama MCP tools to help the user build or update an app for Bahama.

Follow the real Bahama contract:

- `react-vite` frontend
- optional `hono` backend
- optional D1 via `env.DB`

Use Bahama as the system of action:

- create or load the project
- update the project shape before deploy when backend or D1 is needed
- provision D1 only when required
- request deploy instructions or an upload URL when needed
- upload a valid source bundle
- start the deploy
- poll deploy status to a terminal state when possible

Do not invent credentials, provider-specific runtime config, or unsupported frameworks.
