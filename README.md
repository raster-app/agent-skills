# Raster agent skills

[Agent skills](https://agentskills.io/home) that teach AI coding agents to work
with [Raster](https://raster.app): store, organize, search, and serve a user's
assets over the API, drive Raster from a terminal, and start with no account.

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

| Skill                          | Use it when                                                                                                                                    |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `raster-api`                   | An agent should use the Raster API over MCP or REST: authentication, the upload/tag/search/hand-off loop, and starting with no Raster account. |
| `raster-cli`                   | An agent should drive Raster from a terminal or CI with the `raster` command-line client.                                                      |
| `raster-start-without-account` | You want to give a user a hosted library before they have a Raster account. One call creates an organization, library, and key from an email.  |

`raster-api` ships a `references/` folder (rest, mcp, uploading, organizing,
searching, no-account, errors) that the agent loads on demand.

## Claude Code plugin

This repo is also a Claude Code plugin and its own plugin marketplace. The
plugin bundles the skills in `skills/` and the remote Raster MCP server.
Install it from a Claude Code session (v2.1.275 or later):

```text
/plugin install raster --marketplace raster-app/agent-skills
```

On an earlier version, add the marketplace, then install:

```text
/plugin marketplace add raster-app/agent-skills
/plugin install raster@raster
```

From a shell (v2.1.292 or later), run
`claude plugin install raster --marketplace raster-app/agent-skills`.

The server signs in with OAuth: run `/mcp`, sign in to Raster, and choose the
organization the connection may access. If you already added
`https://mcp.raster.app/` with `claude mcp add`, Claude Code keeps that
connection and `/mcp` lists it in place of the plugin's server.

To get updates as they ship, turn on auto-update for the `raster` marketplace in
`/plugin` → **Marketplaces**.

## Cursor plugin

This repo is also a [Cursor plugin](https://cursor.com/docs/plugins). The
manifest at `.cursor-plugin/plugin.json` bundles the skills in `skills/`
and the remote Raster MCP server in `mcp.json`:

```json
{
  "mcpServers": {
    "raster": { "type": "streamable-http", "url": "https://mcp.raster.app/" }
  }
}
```

The server uses OAuth, so no API key is stored in the plugin. On first use,
Cursor opens a Raster sign-in and you grant one organization (or all of yours).

To try it locally before it is in the Cursor Marketplace, copy this repo into
`~/.cursor/plugins/local/raster` (a folder, not a symlink), run
**Developer: Reload Window**, and check **Customize** for the `raster` MCP
server and the `raster-api`, `raster-cli`, and `raster-start-without-account`
skills.

## Authentication

The `raster-api` skill works over an MCP OAuth connection or with a Raster API
key. A key is scoped to one organization and the libraries it may reach. Create
one in your organization settings, or have an agent create one from an email
with no account (see
[Start without an account](https://raster.app/docs/guides/start-without-an-account)).
Treat a key like a password.

## Docs

- API reference: https://raster.app/docs/api
- Agent runbook: https://raster.app/docs/guides/ai-agents
- Install guide: https://raster.app/docs/api/skills
