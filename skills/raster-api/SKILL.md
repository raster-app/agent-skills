---
name: raster-api
description: Use when storing, organizing, searching, or serving a user's assets with Raster over its MCP or REST API, such as uploading from URLs or files, tagging and describing assets, full-text search, moving assets between libraries, or handing back permanent CDN links. Always use this skill when the user mentions Raster, even for a simple "upload this to Raster". It carries the gotchas that prevent the common mistakes (a no-account API key is returned only once, a new upload reaches lists and search a few seconds late, `index` and `trash` are reserved tags, a 404 means out of scope).
license: MIT
metadata:
  author: raster
  version: "1.0.0"
  homepage: https://raster.app/docs/api/skills
  source: https://github.com/raster-app/agent-skills
---

# Using the Raster API

Raster is a digital asset manager. An agent uploads a user's assets, organizes
and searches them, and hands back permanent CDN URLs over one of two transports:

- MCP at `https://mcp.raster.app/` (Streamable HTTP), for agent clients like
  Claude, Cursor, VS Code, and the OpenAI Responses API.
- REST at `https://api.raster.app`, for any HTTP client.

Both accept a Bearer API key or an OAuth access token. Pick the transport your
client speaks; the model and the loop below are the same on both. Full
reference: `https://raster.app/docs/api`.

## Quickstart

Over MCP, add `https://mcp.raster.app/` as a remote MCP server. Your client runs
the OAuth flow: you sign in and grant one organization, or all of yours when the
client requests access to every organization. For server-to-server, paste an
API key instead. Then:

```json
{ "name": "whoami", "arguments": {} }
{ "name": "upload_assets", "arguments": { "organizationId": "acme-co", "libraryId": "barcelona", "sources": [{ "kind": "url", "url": "https://example.com/photo.jpg" }] } }
```

Over REST, every request carries two headers, and responses use a `{ data }` /
`{ error }` envelope:

```bash
# Resolve scope: your organizationId and the libraries the key can reach
curl https://api.raster.app/me \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-07-08'

# Upload a local file (multipart; repeat -F files=@ for a batch, max 20)
curl -X POST 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-07-08' \
  -F 'files=@/path/to/photo.jpg'
```

Each uploaded asset returns a permanent CDN `url` right away. That link is the
deliverable you hand back to the user.

## Choose your transport

| Your client                                              | Use        |
| -------------------------------------------------------- | ---------- |
| An agent platform that speaks MCP (Claude, Cursor, …)    | **MCP**    |
| You control the HTTP layer, or you're in a shell/CI      | **REST**   |

## Authenticate

MCP clients should connect over OAuth, the recommended credential. Add
`https://mcp.raster.app/` as a remote MCP server; the client discovers the
authorization server, registers itself, and opens a Raster consent page where you
sign in and choose the organizations the connection may access. When the client
requests access to every organization, you grant all of yours (the default) or
narrow to one; otherwise you pick one. With every organization granted, `whoami`
returns an `organizations` list instead of a single `organizationId`. The
connection uses your library access and follows it live, so a changed role or
library takes effect without a reconnect. Revoke it under **Settings → Connected
apps**.

Server-to-server calls use an API key, sent as `Authorization: Bearer
<API_KEY>`. REST accepts either an API key or an OAuth access token in that
header, and setting an asset's approval requires the OAuth token. REST also
requires `Api-Version: 2026-07-08`. A key is scoped to one organization and an
allowlist of libraries, each at **Read** or **Write**. Create and scope keys in
organization settings.

An agent with no account can mint a key from an email. See the
`raster-start-without-account` skill.

Details: `https://raster.app/docs/api/mcp/authentication` (MCP) and
`https://raster.app/docs/api/rest/authentication` (REST).

## The loop

Most jobs follow the same five steps. Map the user's intent to these verbs. Each
step names its MCP tools, which map one-to-one to REST endpoints (see the
references).

1. Resolve scope with `whoami`, which returns the `organizationId` and the
   libraries your credential reaches. Pair that `organizationId` with a
   `libraryId` on every asset-level call. Pick a library by name with
   `list_libraries`.
2. Read with `list_assets`, `search_assets`, `get_asset`, and `list_tags` to
   ground later actions in what already exists (for example, reuse a library's
   tags).
3. Upload with `upload_asset` for one asset or `upload_assets` for a batch. Pass
   a public `http(s)` URL the server fetches, or base64 for local bytes. Set
   `parentId` on `upload_asset` to add the file as a variant of an existing asset.
4. Organize with `tag_assets`, `untag_assets`, and `update_asset_description` to
   make assets findable, `transfer_assets` to move them, `delete_assets` to trash
   them, and `set_asset_approval` to record a review decision (OAuth connections
   only).
5. Hand back each asset's permanent CDN `url` and the library URL
   (`raster.app/<organizationId>/<libraryId>`) to the user.

## Critical gotchas

- Call `whoami` first. Every asset call needs an `organizationId` and
  `libraryId` pair. A library your key can't reach returns 404, the same
  response as a library that doesn't exist. Treat it as out of scope and pick
  one `whoami` listed instead of retrying.
- The `apiKey` from `create_organization` / `POST /libraries` is shown once.
  Persist it the moment you receive it.
- An upload returns the permanent CDN `url` and `id` immediately, and the `url`
  serves right away. The asset appears in lists and search a few seconds later,
  so re-list or re-search before concluding an upload failed.
- `index` and `trash` are reserved tags. Applying them fails with
  `BAD_USER_INPUT`, so choose other words.
- One out-of-scope asset id fails a whole `tag_assets` or `transfer_assets`
  call, and one source that fails validation fails a whole `upload_assets`
  call. Fix the bad item and resend. A file refused for its type, or because it
  can't be opened, is left out while the rest upload, and `responseText` names
  it with the reason.
- `delete_assets` is a soft delete. Assets move to trash, stay recoverable for
  30 days, and are then permanently removed.
- Share the CDN `url`, not a signed or app URL. It is the canonical, permanent
  asset link and needs no publish step.

## What do you need?

| Task                                                | Reference                                            |
| --------------------------------------------------- | ---------------------------------------------------- |
| Connect and authenticate over REST                  | [`references/rest.md`](references/rest.md)            |
| Connect over MCP; the full tool list                | [`references/mcp.md`](references/mcp.md)              |
| Upload assets (URL, base64, multipart; limits)      | [`references/uploading.md`](references/uploading.md)  |
| Tag, describe, transfer, trash                      | [`references/organizing.md`](references/organizing.md) |
| Search and list assets, return CDN links            | [`references/searching.md`](references/searching.md)  |
| Start with no Raster account                        | [`references/no-account.md`](references/no-account.md) |
| Error codes and what to do                          | [`references/errors.md`](references/errors.md)        |

## Common mistakes

| Mistake                                                        | Do this instead                                                       |
| ------------------------------------------------------------- | --------------------------------------------------------------------- |
| Acting on a library before calling `whoami`                    | Resolve scope first; pair `organizationId` + `libraryId` every call.  |
| Treating a 404 as "retry"                                      | A 404 means the key can't reach that library. Pick one `whoami` lists. |
| Concluding an upload failed because it's not in search yet     | New uploads reach lists and search a few seconds late; re-list or re-search. |
| Applying the `index` or `trash` tag                            | They're reserved; choose other words.                                 |
| Dropping the `apiKey` from a no-account create call            | It's shown once; persist it immediately.                              |
| Sending 21+ files in one upload, or 101+ ids to delete         | Batch at 20 uploads / 100 deletes; split larger sets.                 |
| Omitting the `Api-Version` header on REST                      | Always send `Api-Version: 2026-07-08` on REST requests.               |
| Handing the user an app or signed URL                          | Return the asset's permanent CDN `url`.                               |

## Error reference

REST and MCP return identical HTTP statuses and messages for the same coded
error. Over MCP, arguments that fail a tool's input schema are refused before the
tool runs, with a plain-text `-32602` error that has no `code` or `status`. Full
detail: [`references/errors.md`](references/errors.md) and
`https://raster.app/docs/api/rest/errors`.

| Code                                 | HTTP | What it means / do                                                        |
| ------------------------------------ | ---- | ------------------------------------------------------------------------- |
| `UNAUTHENTICATED` / `INVALID_API_KEY` | 401  | Missing or wrong key. Send `Authorization: Bearer <key>`.                 |
| `API_VERSION_REQUIRED`               | 400  | Add `Api-Version: 2026-07-08`.                                            |
| `API_KEY_NOT_AUTHORIZED_FOR_LIBRARY` | 404  | Key can't reach that library. Pick one `whoami` lists.                    |
| `API_KEY_READ_ONLY`                  | 403  | A read-only key tried to create a library. Use a key with Write on a library. |
| `BAD_USER_INPUT`                     | 400  | Invalid body or params, including a reserved tag, a bad email, more than 20 files in an upload, or more than 100 ids in a delete (over MCP these two limits are refused as `-32602` schema errors). Fix and resend; split an oversized batch. |
| `PAYLOAD_TOO_LARGE`                  | 413  | A REST file exceeds the per-file limit, or an MCP request body is over 32 MB. Shrink or split. |
| `STORAGE_LIMIT_EXCEEDED` / `LIBRARY_LOCKED` | 403 | Free-plan cap reached. The user must upgrade.                    |
| `INTERNAL_SERVER_ERROR`              | 500  | Server fault. Safe to retry with backoff.                               |

## Start with no account

An agent holding no key can mint an organization, a starter library, and a usable
key from a user's email in one call: `create_organization` on the MCP
`/anonymous` endpoint, or `POST https://api.raster.app/libraries` over REST (no
`Authorization` header). It returns `organizationId`, `libraryId`, `apiKey`, and
a `claimUrl` the user follows to keep the work. Full recipe:
[`references/no-account.md`](references/no-account.md) and the
`raster-start-without-account` skill.

## Resources

- API reference: `https://raster.app/docs/api`
- Agent runbook (per-transport prompts): `https://raster.app/docs/guides/ai-agents`
- MCP tools: `https://raster.app/docs/api/mcp/tools`
- REST endpoints: `https://raster.app/docs/api/rest/endpoints`
- Errors: `https://raster.app/docs/api/rest/errors`
- Drive Raster from a terminal with the `raster-cli` skill.
