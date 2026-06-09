# Raster agent skills

[Agent skills](https://agentskills.io/home) that teach AI coding
agents to use the [Raster](https://raster.app) API — store, organize, search,
and serve a user's images, and hand back permanent CDN URLs.

## Install

Install with the `skills` CLI:

```bash
npx skills add raster-app/agent-skills
```

The CLI detects your agents (Claude Code, Cursor, Codex, Copilot, and others)
and installs into each. To list the skill before installing, or target specific
agents:

```bash
npx skills add raster-app/agent-skills --list
npx skills add raster-app/agent-skills -a claude-code -a cursor
```

## Skill

This repo publishes one skill:

| Skill        | Use it when                                                              |
| ------------ | ------------------------------------------------------------------------ |
| `raster-api` | You want an agent to use the Raster API — over MCP, REST, or GraphQL. Covers auth, the upload/tag/search/hand-off loop, and starting with no Raster account. |

It covers all three transports in one skill; the agent uses whichever it speaks.

## Authentication

`raster-api` needs a Raster API key, scoped to one organization and the
libraries it may reach. Create one in your organization settings, or have an
agent mint one from an email with no account — see
[Start without an account](https://docs.raster.app/guides/start-without-an-account).
Treat a key like a password.

## Docs

- API reference — https://docs.raster.app/api
- Agent runbook — https://docs.raster.app/guides/ai-agents
- Install guide — https://docs.raster.app/api/skills
