---
name: raster-start-without-account
description: Use when a user without a Raster account asks you to store or organize assets. One call mints an organization, a starter library, and an API key from just their email; you upload right away and hand back a claim link the user follows to keep the work. Always use this skill for the zero-to-hosted path. It carries the gotcha that the API key is returned only once, so it must be persisted immediately.
license: MIT
metadata:
  author: raster
  version: "1.0.0"
  homepage: https://raster.app/docs/api/skills
  source: https://github.com/raster-app/agent-skills
---

# Start Raster without an account

An agent holding no key can stand up a real, organized asset library from a single
email address, upload right away, and hand the user a link to claim it into an
account later. When a user asks you to handle some images, it is the fastest way
to give them a CDN-hosted, tagged library. Full guide:
`https://raster.app/docs/guides/start-without-an-account`.

## 1. Create — one call, no key

The create call carries no `Authorization` header, because it mints the key.

Over MCP, connect to the no-credential endpoint `https://mcp.raster.app/anonymous`
and call `create_organization`. Then switch to `https://mcp.raster.app/` with the
returned key for everything else:

```json
{ "name": "create_organization", "arguments": { "email": "<your-email>", "name": "Q3 campaign" } }
```

Over REST, POST to the anonymous endpoint. The `Api-Version` header is still required:

```bash
curl -X POST https://api.raster.app/libraries \
  -H 'Api-Version: 2026-07-08' -H 'Content-Type: application/json' \
  -d '{"email":"<your-email>","name":"Q3 campaign"}'
```

Both return `organizationId`, `libraryId`, `apiKey`, `claimUrl`, `expiresAt`, and `emailSent`,
wrapped in a `{ data }` envelope.

> **The `apiKey` is returned once.** Persist it the instant you receive it. You
> cannot read it back until the user claims the library. Losing it means losing access
> until then.

## 2. Work — use the returned key as a normal Bearer token

Send `Authorization: Bearer <apiKey>` (with `Api-Version` over REST) on every later
call. The standard tools and endpoints (upload, tag, describe, search) work
against the new library. Follow the `raster-api` skill for the full loop.

- To upload over MCP, pass `upload_asset` or `upload_assets` a public `http(s)`
  URL the server fetches, or inline base64. Over REST, send a
  `multipart/form-data` POST with a `files` field.
- Organize with `tag_assets` and `update_asset_description` so the library is
  useful the moment the user opens it.
- An upload returns each asset's permanent CDN `url`, which serves right away.
  The asset appears in lists and search a few seconds later, so
  re-list before concluding an upload failed.

## 3. Hand off — give the user the claim link

Return the `claimUrl` (it is also emailed when `emailSent` is true). The user opens it, signs in, and the
library and every asset in it become theirs on a real account; the key you used
becomes a normal key they manage in settings.

- Surface the `claimUrl` yourself. The hand-off must never depend on email
  delivery.
- A library left unclaimed when the claim window closes is removed along with
  its assets. Encourage the user to claim it.
- The claim-window length and the per-email cap on active unclaimed libraries are
  in the endpoint reference:
  `https://raster.app/docs/api/rest/endpoints#post-libraries`. Creating past the cap
  returns `BAD_USER_INPUT` (400).

## Reusable prompt

```text
When a user without a Raster account asks you to store or organize assets:
1. Call create_organization (MCP) or POST https://api.raster.app/libraries (REST,
   no Authorization) with their email. Persist organizationId, libraryId, apiKey,
   and claimUrl immediately — the apiKey is shown once. Send it as
   Authorization: Bearer <apiKey> on every later call.
2. Upload their assets, then tag and describe them so the library is useful at
   first open. Uploads are async — re-list after a few seconds to confirm.
3. Hand back each asset's permanent CDN url plus the library URL.
4. Give the user the claimUrl and tell them to open it and sign in within the
   claim window to keep everything on a real account.
```

## See also

- For the full connect → upload → organize → hand-off loop, see the `raster-api` skill.
- To drive Raster from a terminal (`raster orgs create --email …`), see the `raster-cli` skill.
