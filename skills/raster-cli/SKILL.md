---
name: raster-cli
description: Use when driving Raster from a terminal, a shell script, or a CI pipeline with the `raster` command-line client — sign in over OAuth or an API key, then list, search, upload, download, tag, describe, transfer, or delete assets, with `--json` for machine-readable output. Always use this skill for terminal or CI work with Raster; it carries the credential-resolution order, the `--library` disambiguation, and the stable exit codes that scripts depend on.
license: MIT
metadata:
  author: raster
  version: "1.0.0"
  homepage: https://raster.app/docs/api/skills
  source: https://github.com/raster-app/agent-skills
---

# Raster CLI

The Raster CLI (`@raster-app/cli`, command `raster`) is a command-line client for
the Raster REST API. It covers the full asset workflow — list, search, upload,
download, tag, transfer, delete — from a terminal or a CI pipeline. It is in
**Alpha**, so commands and flags may change before 1.0. Full reference:
`https://raster.app/docs/api/cli`.

## Install

```sh
npm install -g @raster-app/cli
```

Run once without installing: `npx -p @raster-app/cli raster whoami`.

## Authenticate

Two paths — pick by context:

```sh
# Interactive (a person at a terminal): OAuth, tied to your user account.
raster auth login          # browser when available; device code over SSH/headless
raster auth status         # show the active credential
raster auth logout         # revoke + clear

# CI and scripts: an organization API key.
export RASTER_API_KEY=pk_…  # or pass --api-key pk_… per command
```

The active credential resolves in order: `--api-key` → `RASTER_API_KEY` → a stored
OAuth token → a config-file key. An ambient `RASTER_API_KEY` therefore beats a
stored OAuth session — handy for scripts.

## Org and library

A key carries its organization and libraries, so `--org` is optional and
`--library` is optional when the key has a single library. Pass `--library <id>`
when the key can reach several; a broad user-scope OAuth login reaches every org
you belong to, so pass `--org <id>` (or set `RASTER_ORG`) to choose one. Run
`raster whoami` to list what the current credential reaches.

## Common tasks

```sh
# What can this credential reach?
raster whoami

# Upload local files (batched at 20 per request), then confirm
raster assets upload ./photos/*.png --library brand
raster assets ls --library brand --tag product

# Find images and get their ids (pipeable with --json)
raster assets search "golden hour" --json | jq '.hits[].id'

# Organize
raster tags add asset_123 asset_456 --tag launch --library brand
raster assets describe asset_123 --text "Hero shot" --library brand
raster assets transfer asset_123 --to archive --library brand

# Retire (soft delete — recoverable from trash)
raster assets rm asset_123 --yes --library brand

# No account yet: mint an org + library + key from an email; prints a claim URL
raster orgs create --email user@example.com --save
```

## Scripting

Human-readable tables go to **stdout**; progress and notes go to **stderr**, so
stdout stays pipeable. Add `--json` for the raw API payload; `--verbose` logs each
request to stderr. Exit codes are **stable across releases** — branch on them:

| Code | Meaning                        |
| ---- | ------------------------------ |
| 0    | Success                        |
| 2    | Usage error (bad flags/args)   |
| 3    | Authentication (missing/rejected key) |
| 4    | Not found                      |
| 5    | Validation or conflict         |
| 6    | File too large                 |
| 7    | Network failure                |

## Gotchas

- **Alpha** — pin a version in CI; commands/flags may change before 1.0.
- **Asset ids are positional**; `--library` selects the library when the key
  reaches more than one. `raster <command> --help` lists every flag.
- **Uploads process asynchronously** — an uploaded asset appears in `assets ls`
  and search a few seconds later. Re-list before concluding an upload failed.
- **`raster assets rm` is a soft delete** — assets go to trash and stay
  recoverable until permanently removed.
- **`index` and `trash` are reserved tags** — `tags add` rejects them.

## Environment variables

- `RASTER_API_KEY` — API key (above a stored OAuth token, below `--api-key`).
- `RASTER_ORG` — default organization id for an OAuth login when `--org` is omitted.
- `RASTER_CONFIG_HOME` — config directory (default `~/.config/raster`).

## See also

- Use the API directly (MCP / REST / GraphQL) — the `raster-api` skill.
- Start with no account — the `raster-start-without-account` skill.
