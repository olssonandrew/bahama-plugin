# Bahama Plugin

[Bahama](https://www.bahama.ai) is the agent-first cloud platform for getting AI-built apps live on the web. Your coding agent can already write the app; Bahama gives it the infrastructure tools to create projects, provision resources, test against managed services, and deploy to a live URL without asking you to juggle cloud accounts, credentials, or deployment consoles.

Install Bahama in the coding tool you already use, then ask your agent to build and ship. Bahama handles the last mile: hosted deployments, managed project resources, databases, storage, secrets, and the orchestration needed to turn local code into a working web app.

Bahama currently supports React apps with Vite frontends, static sites, Hono backend APIs, backend-only Hono APIs, and Bahama-managed database access from server-side code. For full-stack apps, the default pattern is a Vite frontend with a Workers-compatible Hono backend. Support for additional frameworks is coming soon.

## This Plugin

This repository packages two components into a standard AI plugin that works across agentic coding tools:

- **Bahama MCP server connection**: connects your agent to Bahama at `https://www.bahama.ai/api/mcp`.
- **Bahama Builder skill**: teaches the agent how to build, test, provision, package, and deploy apps using Bahama's supported app contracts.

Together, the MCP server is the action layer and the skill is the operating guide. The agent uses the skill to make good implementation choices, then uses Bahama MCP tools to create projects, provision resources, and deploy.

## Install

Bahama installs as a standard plugin in agentic coding tools such as Claude Code, Cursor, Codex, and other MCP-compatible clients. Once installed, your agent can authenticate with Bahama and call the hosted MCP server directly from your normal coding workflow.

For tool-specific installation steps, see the official plugin documentation:

- [Claude Code plugin installation](https://code.claude.com/docs/en/discover-plugins)
- [Cursor Marketplace](https://cursor.com/plugins)
- [Codex plugin documentation](https://developers.openai.com/codex/plugins/build)

## Getting Started

Once Bahama is installed, simply ask your agent to build something and deploy it with Bahama. The agent will use the Bahama skill for guidance and the Bahama MCP server to create projects, provision resources, and publish the app.

Try prompts like:

- "Use Bahama to build and deploy a notes app with a database."
- "Build a recipe app with saved shopping lists and ship it on Bahama."
- "Create a small customer portal, add persistent storage, and deploy it with Bahama."
- "Take this existing Vite app, add a backend API, and publish it through Bahama."
- "Deploy this project on Bahama and fix anything blocking the build."

## Authentication

Bahama MCP uses OAuth. Your agent tool should prompt you to sign in to Bahama.ai and approve access when the plugin first needs to use Bahama MCP tools.

## Learn More

Visit [bahama.ai](https://www.bahama.ai) for the product overview and early access.
