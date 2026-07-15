# Raster agent skills

[Agent skills](https://agentskills.io/home) that teach AI coding agents to work
with [Raster](https://raster.app) — store, organize, search, and serve a user's
images over the API, drive it from a terminal, and start with no account.

## Install

Install with the `skills` CLI:

```bash
npx skills add raster-app/agent-skills
```

The CLI detects your agents (Claude Code, Cursor, Codex, Copilot, and others)
and installs into each. To list the skills before installing, install just one,
or target specific agents:

```bash
npx skills add raster-app/agent-skills --list
npx skills add raster-app/agent-skills --skill raster-api
npx skills add raster-app/agent-skills -a claude-code -a cursor
```

## Skills

| Skill                          | Use it when                                                                                                                                        |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| `raster-api`                   | An agent should use the Raster API over MCP, REST, or GraphQL — auth, the upload/tag/search/hand-off loop, and starting with no Raster account.    |
| `raster-cli`                   | An agent should drive Raster from a terminal or CI with the `raster` command-line client.                                                          |
| `raster-start-without-account` | You want to give a user a hosted image library before they have a Raster account — one call mints an organization, library, and key from an email. |

`raster-api` ships a `references/` folder (rest, mcp, graphql, uploading,
organizing, searching, no-account, errors) that the agent loads on demand.

## Authentication

The API skills need a Raster API key, scoped to one organization and the
libraries it may reach. Create one in your organization settings, or have an
agent mint one from an email with no account — see
[Start without an account](https://docs.raster.app/guides/start-without-an-account).
Treat a key like a password.

## Docs

- API reference — https://docs.raster.app/api
- Agent runbook — https://docs.raster.app/guides/ai-agents
- Install guide — https://docs.raster.app/api/skills
