---
name: raster-start-without-account
description: Use when a user without a Raster account asks you to store or organize images. One call mints an organization, a starter library, and an API key from just their email; you upload right away and hand back a claim link the user follows to keep the work. Always use this skill for the zero-to-hosted path — it carries the gotcha that the API key is returned only once, so it must be persisted immediately.
license: MIT
metadata:
  author: raster
  version: "1.0.0"
  homepage: https://raster.app/docs/api/skills
  source: https://github.com/raster-app/agent-skills
---

# Start Raster without an account

An agent holding no key can stand up a real, organized image library from a single
email address, upload right away, and hand the user a link to claim it into an
account later. This is the fastest path from "a user asked me to handle some
images" to "here is your CDN-hosted, tagged library." Full guide:
`https://raster.app/docs/guides/start-without-an-account`.

## 1. Create — one call, no key

The create call carries **no `Authorization` header** — it mints the key.

**MCP** — connect to the no-credential endpoint `https://mcp.raster.app/anonymous`
and call `create_organization` (then switch to `https://mcp.raster.app/` with the
returned key for everything else):

```json
{ "name": "create_organization", "arguments": { "email": "user@example.com", "name": "Q3 campaign" } }
```

**REST** — POST to the anonymous endpoint (the `Api-Version` header is still required):

```bash
curl -X POST https://api.raster.app/libraries \
  -H 'Api-Version: 2026-05-20' -H 'Content-Type: application/json' \
  -d '{"email":"user@example.com","name":"Q3 campaign"}'
```

Both return `organizationId`, `libraryId`, `apiKey`, and `claimUrl` (REST wraps it
in the `{ data }` envelope).

> **The `apiKey` is returned once** and stored only encrypted afterward. Persist it
> the instant you receive it — you cannot read it back until the user claims the
> library. Losing it means starting over.

## 2. Work — use the returned key as a normal Bearer token

Send `Authorization: Bearer <apiKey>` (with `Api-Version` over REST) on every later
call. Every standard tool and endpoint works against the new library — upload,
tag, describe, search. Follow the `raster-api` skill for the full loop.

- **Upload** — `upload_asset` / `upload_assets` over MCP take a public `http(s)`
  URL the server fetches, or inline base64; over REST it's a `multipart/form-data`
  POST with a `files` field.
- **Organize** — `tag_assets` and `update_asset_description` so the library is
  useful the moment the user opens it.
- Uploads process **asynchronously** — re-list a few seconds later before
  concluding one failed. Each asset returns a permanent CDN `url`.

## 3. Hand off — give the user the claim link

Return the `claimUrl` (it is also emailed). The user opens it, signs in, and the
library and every asset in it become theirs on a real account; the key you used
becomes a normal key they manage in settings.

- **Surface the `claimUrl` yourself** — the hand-off must never depend on email
  delivery.
- A library left **unclaimed** when the claim window closes is removed along with
  its images. Encourage the user to claim it.
- The claim-window length and the per-email cap on active unclaimed libraries are
  on the canonical endpoint:
  `https://raster.app/docs/api/rest/endpoints#post-libraries`. Creating past the cap
  returns `BAD_USER_INPUT` (400).

## Reusable prompt

```text
When a user without a Raster account asks you to store or organize images:
1. Call create_organization (MCP) or POST https://api.raster.app/libraries (REST,
   no Authorization) with their email. Persist organizationId, libraryId, apiKey,
   and claimUrl immediately — the apiKey is shown once. Send it as
   Authorization: Bearer <apiKey> on every later call.
2. Upload their images, then tag and describe them so the library is useful at
   first open. Uploads are async — re-list after a few seconds to confirm.
3. Hand back each asset's permanent CDN url plus the library URL.
4. Give the user the claimUrl and tell them to open it and sign in within the
   claim window to keep everything on a real account.
```

## See also

- The full connect → upload → organize → hand-off loop — the `raster-api` skill.
- Drive Raster from a terminal (`raster orgs create --email …`) — the `raster-cli` skill.
