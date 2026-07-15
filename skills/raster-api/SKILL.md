---
name: raster-api
description: Use when storing, organizing, searching, or serving a user's images with Raster over its API — uploading from URLs or files, tagging and describing assets, full-text search, moving assets between libraries, or handing back permanent CDN links, over MCP, REST, or GraphQL. Always use this skill when the user mentions Raster, even for a simple "upload this to Raster" — it carries the gotchas (the API key is shown once, uploads process asynchronously, dedupe before uploading, `index`/`trash` are reserved tags, a 404 means out-of-scope) that prevent the common mistakes.
license: MIT
metadata:
  author: raster
  version: "1.0.0"
  homepage: https://raster.app/docs/api/skills
  source: https://github.com/raster-app/agent-skills
---

# Using the Raster API

Raster is a digital asset manager for images. An agent uploads a user's images,
organizes and searches them, and hands back permanent CDN URLs — over one of
three transports, all authenticated with the same Bearer API key:

- **MCP** — `https://mcp.raster.app/` (Streamable HTTP). For agent clients like
  Claude, Cursor, VS Code, and the OpenAI Responses API.
- **REST** — `https://api.raster.app`. Any HTTP client.
- **GraphQL** — `https://api.raster.app/`. Same surface, one POST endpoint.

Pick the transport your client speaks; the model and the loop below are identical
across all three. Full reference: `https://raster.app/docs/api`.

## Quickstart

**MCP** — point your client at `https://mcp.raster.app/` with a Bearer key, then:

```json
{ "name": "whoami", "arguments": {} }
{ "name": "upload_assets", "arguments": { "organizationId": "org_…", "libraryId": "lib_…", "sources": [{ "kind": "url", "url": "https://example.com/photo.jpg" }] } }
```

**REST** — every request carries two headers; responses use a `{ data }` / `{ error }` envelope:

```bash
# Resolve scope: your organizationId and the libraries the key can reach
curl https://api.raster.app/me \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20'

# Upload a local file (multipart; repeat -F files=@ for a batch, max 20)
curl -X POST 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20' \
  -F 'files=@/path/to/photo.jpg'
```

Each uploaded asset returns a permanent CDN `url` right away — that link is the
deliverable you hand back to the user.

## Choose your transport

| Your client                                              | Use        |
| -------------------------------------------------------- | ---------- |
| An agent platform that speaks MCP (Claude, Cursor, …)    | **MCP**    |
| You control the HTTP layer, or you're in a shell/CI      | **REST**   |
| You already query a GraphQL layer                        | **GraphQL** |

## Authenticate

Send `Authorization: Bearer <API_KEY>` on every call; REST also requires
`Api-Version: 2026-05-20`. A key is scoped to **one organization** and
an **allowlist of libraries**, each at a **read** or **write** access level.
Create and scope keys in organization settings. An agent with no key can mint
one from an email — see the `raster-start-without-account` skill.

Details: `https://raster.app/docs/api/rest/authentication`.

## The loop

Most jobs follow the same five steps. Map the user's intent to these verbs; each
names its MCP tool (the REST endpoint and GraphQL field are one-to-one — see the
references).

1. **Resolve scope** — `whoami` returns the `organizationId` and the libraries
   your key reaches. Pair that `organizationId` with a `libraryId` on every
   asset-level call. Pick a library by name with `list_libraries`.
2. **Read and dedupe** — `list_assets`, `search_assets`, `get_asset`, `list_tags`
   to see what already exists before you write.
3. **Upload** — `upload_asset` for one image, `upload_assets` for a batch. Pass a
   public `http(s)` URL the server fetches, or base64 for local bytes.
4. **Organize** — `tag_assets`, `untag_assets`, `update_asset_description` to make
   assets findable; `transfer_assets` to move them; `delete_assets` to trash them.
5. **Hand back** — return each asset's permanent CDN `url` and the library URL
   (`raster.app/<organizationId>/<libraryId>`) to the user.

## Critical gotchas

- **Call `whoami` first.** Every asset call needs an `organizationId` + `libraryId`
  pair. A library your key can't reach returns **404** — treat it as out of scope
  and pick one `whoami` listed. (404 hides existence; it is not "try again".)
- **The `apiKey` from `create_organization` / `POST /libraries` is shown once.**
  Persist it the instant you receive it — it is stored only encrypted afterward.
- **Uploads are asynchronous.** The permanent CDN `url` and `id` come back
  immediately, but the asset appears in lists and search a few seconds later.
  Re-list or re-search before concluding an upload failed.
- **Dedupe before uploading.** Run `search_assets` or `list_assets` and skip
  images already present — uploads are not deduplicated for you.
- **`index` and `trash` are reserved tags** — applying them fails with
  `BAD_USER_INPUT`. Choose other words.
- **A batch is all-or-nothing.** One invalid file or one out-of-scope asset id
  fails the whole `upload_assets` / `tag_assets` / `transfer_assets` call. Fix the
  bad item and resend.
- **`delete_assets` is a soft delete** — assets move to trash and stay
  recoverable until permanently removed.
- **Share the CDN `url`, not a signed or app URL.** It is the canonical, permanent
  image link, production-ready with no publish step.

## What do you need?

| Task                                                | Reference                                            |
| --------------------------------------------------- | ---------------------------------------------------- |
| Connect and authenticate over REST                  | [`references/rest.md`](references/rest.md)            |
| Connect over MCP; the full tool list                | [`references/mcp.md`](references/mcp.md)              |
| Query over GraphQL                                  | [`references/graphql.md`](references/graphql.md)      |
| Upload images (URL, base64, multipart; limits)      | [`references/uploading.md`](references/uploading.md)  |
| Tag, describe, transfer, trash                      | [`references/organizing.md`](references/organizing.md) |
| Search and list assets, return CDN links            | [`references/searching.md`](references/searching.md)  |
| Start with no Raster account                        | [`references/no-account.md`](references/no-account.md) |
| Every error code and what to do                     | [`references/errors.md`](references/errors.md)        |

## Common mistakes

| Mistake                                                        | Do this instead                                                       |
| ------------------------------------------------------------- | --------------------------------------------------------------------- |
| Acting on a library before calling `whoami`                    | Resolve scope first; pair `organizationId` + `libraryId` every call.  |
| Treating a 404 as "retry"                                      | A 404 means the key can't reach that library — pick one `whoami` lists. |
| Concluding an upload failed because it's not in search yet     | Uploads are async; re-list/re-search after a few seconds.             |
| Uploading without checking for duplicates                      | `search_assets` / `list_assets` first; skip what exists.              |
| Applying the `index` or `trash` tag                            | They're reserved; choose other words.                                 |
| Dropping the `apiKey` from a no-account create call            | It's shown once — persist it immediately.                             |
| Sending 21+ files in one upload, or 101+ ids to delete         | Batch at 20 uploads / 100 deletes; split larger sets.                 |
| Omitting the `Api-Version` header on REST                      | Always send `Api-Version: 2026-05-20` on REST requests.               |
| Handing the user an app or signed URL                          | Return the asset's permanent CDN `url`.                               |

## Error reference

REST and GraphQL return **identical** HTTP statuses and messages. Full detail:
[`references/errors.md`](references/errors.md) and
`https://raster.app/docs/api/rest/errors`.

| Code                                 | HTTP | What it means / do                                                        |
| ------------------------------------ | ---- | ------------------------------------------------------------------------- |
| `UNAUTHENTICATED` / `INVALID_API_KEY` | 401  | Missing/wrong key. Send `Authorization: Bearer <key>`.                    |
| `API_VERSION_REQUIRED`               | 400  | Add `Api-Version: 2026-05-20`.                                            |
| `API_KEY_NOT_AUTHORIZED_FOR_LIBRARY` | 404  | Key can't reach that library — pick one `whoami` lists.                  |
| `API_KEY_READ_ONLY`                  | 403  | A read-only key tried to create a library. Use a key with Write on a library. |
| `BAD_USER_INPUT`                     | 400  | Invalid body/params (incl. reserved tags, bad email). Fix and resend.    |
| `MAX_ASSETS_EXCEEDED`                | 400  | >20 files uploaded or >100 ids deleted. Split the batch.                 |
| `PAYLOAD_TOO_LARGE`                  | 413  | A file exceeds the per-file limit. Shrink or split.                      |
| `STORAGE_LIMIT_EXCEEDED` / `LIBRARY_LOCKED` | 403 | Free-plan cap reached. The user must upgrade.                    |
| `INTERNAL_SERVER_ERROR`              | 500  | Server fault — safe to retry with backoff.                              |

## Start with no account

An agent holding no key can mint an organization, a starter library, and a usable
key from a user's email in one call — `create_organization` over MCP, or
`POST https://api.raster.app/libraries` over REST (no `Authorization` header). It
returns `organizationId`, `libraryId`, `apiKey`, and a `claimUrl` the user follows
to keep the work. Full recipe: [`references/no-account.md`](references/no-account.md)
and the `raster-start-without-account` skill.

## Resources

- API reference — `https://raster.app/docs/api`
- Agent runbook (per-transport prompts) — `https://raster.app/docs/guides/ai-agents`
- MCP tools — `https://raster.app/docs/api/mcp/tools`
- REST endpoints — `https://raster.app/docs/api/rest/endpoints`
- Errors — `https://raster.app/docs/api/rest/errors`
- Drive Raster from a terminal — the `raster-cli` skill
