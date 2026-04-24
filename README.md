# Bahama Plugin

Bahama for Claude Code packages Bahama's hosted MCP server together with a Bahama-specific build and deploy skill. It lets Claude create projects, provision D1 when needed, and deploy supported Bahama apps through the existing Bahama control plane.

## What it includes

- a hosted MCP connection to `https://bahama.ai/api/mcp`
- the `bahama-builder` skill

## Private testing

### Load directly during development

```bash
claude --plugin-dir ./plugins/bahama
```

### Install from the local marketplace

```bash
/plugin marketplace add ./plugins/bahama
/plugin install bahama@bahama
/reload-plugins
```

## Authentication

Bahama MCP uses OAuth. Users should connect Bahama through Claude Code and complete the Bahama sign-in and consent flow when prompted.

This plugin does not require users to paste an API key for normal MCP use.

The current staged plugin is pointed at the Bahama dev build for private testing. Swap the URL in `.mcp.json` to the production host before public release.

## Supported Bahama app contract

- `react-vite` frontend
- optional `hono` backend
- optional D1 storage exposed to app code as `env.DB`

The deployable app bundle must include:

- `package.json`
- one supported lockfile
- `src/`
- `index.html`

Hono backends must use a Workers-compatible `server/index.*` entry and must not use Node server adapters such as `@hono/node-server`.

## Example prompts

- `Build a notes app on Bahama with React + Vite, a Hono API, and D1 storage.`
- `Create a new Bahama project for a recipe app and deploy it.`
- `Update my Bahama app to use a Hono backend and then redeploy it.`
