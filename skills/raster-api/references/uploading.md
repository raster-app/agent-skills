# Uploading assets

Raster ingests an asset three ways; pick the one your source and transport allow:

| You have                          | Use                                                  |
| --------------------------------- | ---------------------------------------------------- |
| A public `http(s)` URL            | MCP `url` source (the server fetches it).            |
| Local bytes over MCP              | inline base64 `source`.                              |
| Local files over REST             | `multipart/form-data`.                               |

Every path returns each asset's permanent CDN `url` and `id` right away. That
`url` is the canonical link you hand back.

## Limits and semantics

- At most 20 files per request (`upload_assets` / multipart).
- REST multipart: a file over the per-file size limit fails the whole request
  with `413 PAYLOAD_TOO_LARGE`.
- MCP: every source is checked before any file is stored. The whole call fails
  when a source fails validation (its schema, or a URL that resolves to a private
  network), a URL can't be fetched, or the request body is over 32 MB. Fix the
  cause and resend. A URL file over 1 GB is left out and named in
  `responseText` while the rest upload.
- Over either transport, a file refused for its type, or because it can't be
  opened, is left out while the rest of the batch uploads; `responseText` names
  each refused file and why. If no file is accepted, the call fails with
  `BAD_USER_INPUT`.
- The `url` serves as soon as the upload returns. The asset appears in lists and
  search a few seconds later, so re-list before concluding an upload failed.

## Recipes

REST: multipart, repeating `files` (let the client set the multipart `Content-Type`):

```bash
curl -X POST 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-07-08' \
  -F 'files=@/path/a.jpg' -F 'files=@/path/b.png'
```

MCP: `upload_assets` with a `sources[]` array. Each source is a URL or base64
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

## Upload a variant of an existing asset

To attach an upload as a variant of an existing asset instead of creating a new
one, pass that asset's id as `parentId`. Exactly one file is allowed, and the
parent must be an original asset in the same library. The returned asset carries
that `parentId` and an `appUrl` that opens the parent with the variant selected;
it is nested under the parent's `variants`.

REST: add a `parentId` form field:

```bash
curl -X POST 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-07-08' \
  -F 'files=@/path/to/variant.jpg' \
  -F 'parentId=<PARENT_ASSET_ID>'
```

MCP: pass `parentId` to `upload_asset`:

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

Full detail for REST: `https://raster.app/docs/api/rest/endpoints`. MCP tools:
`https://raster.app/docs/api/mcp/tools`.
