# Uploading images

Raster ingests an image three ways; pick the one your source and transport allow:

| You have                          | Use                                                  |
| --------------------------------- | ---------------------------------------------------- |
| A public `http(s)` URL            | MCP/GraphQL `url` source — the server fetches it.    |
| Local bytes over MCP              | inline base64 `source`.                              |
| Local files over REST or GraphQL  | `multipart/form-data`.                               |

Every path returns each asset's permanent CDN `url` and `id` right away — that
`url` is the canonical link you hand back.

## Limits and semantics

- **Max 20 files per request** (`upload_assets` / multipart / `uploadAssets`).
- **All-or-nothing.** Every source is validated (schema, address safety, size)
  before any byte is stored; one bad source fails the whole batch. Fix it and resend.
- **Asynchronous.** `url` and `id` come back immediately, but the asset appears in
  lists and search a few seconds later once processing finishes. Re-list before
  concluding an upload failed — the `url` is permanent and goes live when
  processing completes.

## Recipes

REST — multipart, repeat `files` (let the client set the multipart `Content-Type`):

```bash
curl -X POST 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20' \
  -F 'files=@/path/a.jpg' -F 'files=@/path/b.png'
```

MCP — `upload_assets` with a `sources[]` array; each source is a URL or base64
payload, and the two may be mixed:

```json
{
  "name": "upload_assets",
  "arguments": {
    "organizationId": "<orgId>",
    "libraryId": "<libraryId>",
    "sources": [
      { "kind": "url", "url": "https://example.com/a.jpg" },
      { "kind": "base64", "filename": "shot.png", "mimeType": "image/png", "data": "iVBORw0KGgo..." }
    ]
  }
}
```

Use `upload_asset` with a single `source` for one file. The base64 variant
requires `filename` and `mimeType`; the `url` variant must be an `http(s)` URL.
Over GraphQL, `uploadAssets` takes `files: [Upload!]!` plus `email` and returns
`assets { id url }`.

## Upload a variant of an existing asset

Attach an upload as a **variant** of an existing asset instead of creating a new
one, by passing its id as `parentId`. Exactly one file
is allowed, and the parent must be an original asset in the same library. The
returned asset carries that `parentId` and an `appUrl` that opens the parent with
the variant selected; it is nested under the parent's `views`.

REST — add a `parentId` form field:

```bash
curl -X POST 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20' \
  -F 'files=@/path/to/variant.jpg' \
  -F 'parentId=<PARENT_ASSET_ID>'
```

MCP — pass `parentId` to `upload_asset`:

```json
{
  "name": "upload_asset",
  "arguments": {
    "organizationId": "<orgId>",
    "libraryId": "<libraryId>",
    "parentId": "<PARENT_ASSET_ID>",
    "source": { "kind": "url", "url": "https://example.com/variant.jpg" }
  }
}
```

Full detail — REST: `https://raster.app/docs/api/rest/endpoints`. MCP tools:
`https://raster.app/docs/api/mcp/tools`.
