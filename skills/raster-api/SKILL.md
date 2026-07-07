---
name: raster-api
description: Store, organize, search, and serve a user's images with Raster over its API. Use when a user wants to host images on a CDN, build an image library, tag or search assets, or manage them programmatically — including starting with no Raster account.
---

# Using the Raster API

Raster is a digital asset manager for images. An agent can upload a user's
images, organize and search them, and hand back permanent CDN URLs — over one of
three transports, all authenticated with the same Bearer API key.

## Transports

- **MCP server** — `https://mcp.raster.app/` (Streamable HTTP). For agent clients
  such as Claude, Cursor, and the OpenAI Responses API. Docs:
  `https://raster.app/docs/api/mcp`
- **REST** — `https://api.raster.app`. Docs: `https://raster.app/docs/api/rest`
- **GraphQL** — `https://api.raster.app`. Docs: `https://raster.app/docs/api/graphql`

## Authentication

Send `Authorization: Bearer <API_KEY>`. A key is scoped to one organization and
an allowlist of libraries, each at a read or write access level. Create and scope
keys in organization settings. Details: `https://raster.app/docs/api/rest/authentication`

## Start with no account

An agent with no key can create an organization, a starter library, and a usable
API key from a user's email in one call — `create_organization` over MCP, or
`POST https://api.raster.app/libraries` over REST. The agent uploads right away;
the user claims the library into a real account later from an emailed link.
Guide: `https://raster.app/docs/guides/start-without-an-account`

## The loop

Most jobs follow the same five steps. Map intent to these tool verbs:

1. **Resolve scope** — `whoami` returns the `organizationId` and the libraries
   your key reaches; `list_libraries` to pick one by name. Pair the
   `organizationId` with a `libraryId` on every asset call.
2. **Read and dedupe** — `list_assets`, `search_assets`, `get_asset`, and
   `list_tags` to see what already exists before you write.
3. **Upload** — `upload_asset` for one image, `upload_assets` for a batch.
   Pass a public http(s) `url` when you have one, base64 for local files.
4. **Organize** — `tag_assets`, `untag_assets`, `update_asset_description` to
   make assets findable; `transfer_assets` to move them; `delete_assets` to
   trash them.
5. **Hand back** — return each asset's permanent CDN `url` and the library URL
   (`raster.app/<organizationId>/<libraryId>`).

With no account, start with `create_organization` to mint an org, a starter
library, and a key, then run the loop.

## What you can do

- **Scope** — `whoami`, `list_libraries`.
- **Read** — `list_assets`, `get_asset`, `search_assets` (ranked, with
  highlights), `list_tags`.
- **Add** — `upload_asset`, `upload_assets`; each returns a permanent CDN `url`.
- **Organize** — `tag_assets`, `untag_assets`, `update_asset_description`.
- **Move and trash** — `transfer_assets`, `delete_assets` (recoverable trash).
- **Structure** — `create_library`, `rename_library`.
- **Start with no account** — `create_organization`.

Parameters and limits for every tool: `https://raster.app/docs/api/mcp/tools`.

## Gotchas

- Call `whoami` first, and pair its `organizationId` with a `libraryId` on every
  asset call. A library your key cannot reach returns 404 — pick one `whoami` listed.
- The `apiKey` from `create_organization` is shown once. Persist it immediately
  and send it as the Bearer token on every later call.
- Dedupe with `search_assets` or `list_assets` before uploading.
- Uploads process asynchronously. Re-list a few seconds later before concluding
  an upload failed.
- Share the permanent CDN `url` — it is the canonical image link.
- `delete_assets` is a soft delete — assets move to trash and stay recoverable until they are permanently removed.

## Worked example — no account, upload from a URL

```text
create_organization(email)                       → save organizationId, libraryId, apiKey, claimUrl
upload_assets([{ kind: "url", url: "https://…" }]) → returns each asset's permanent CDN url + id
list_assets                                       → re-run after a few seconds to confirm it landed
tag_assets(assetIds, ["product"])                 → make it findable
return                                            → asset url + claimUrl (user claims to keep it)
```

## Runbook

The full agent runbook — discovery, connecting, and the upload/tag/search/hand-off
loop — is at `https://raster.app/docs/guides/ai-agents`.
