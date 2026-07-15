# Searching and listing assets

Three read verbs find assets and return the permanent CDN `url` you hand back.

## search_assets — across libraries

`search_assets` (REST `GET /organizations/:orgId/search/assets?q=`, GraphQL
`searchAssets`) runs a full-text query across every library the key authorizes,
matching `name`, `description`, and `tags`. Hits come back relevance-ranked with
highlighted snippets.

- Empty or whitespace `q` returns recent assets in scope (no ranking) — a quick
  "what's in here?".
- Repeat the `libraries` param (REST `?libraries=a,b`) to narrow to a subset. Naming
  a library the key can't reach fails the whole call.

```bash
curl 'https://api.raster.app/organizations/<orgId>/search/assets?q=golden%20hour&libraries=<libraryId>' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20'
```

## list_assets — one library

`list_assets` (REST `GET .../libraries/:libraryId/assets`, GraphQL `assets`) pages
through a single library. Filter with `tags` (max 5) to return only assets carrying
them; paginate with `page` / `pageSize`.

## get_asset — one by id

`get_asset` (REST `GET .../assets/:assetId`, GraphQL `asset`) returns a single asset
by the `id` from a list, search, or upload.

## What to return

Each asset carries a permanent CDN `url` — the canonical, production-ready image
link. Hand that `url` to the user. Pass its `tags` and `description` back as context,
and point to the library at `raster.app/<organizationId>/<libraryId>`.

Recently uploaded assets appear in search and lists a few seconds after upload
(processing is asynchronous), so re-run the query before concluding an asset is
missing.

Full reference: `https://raster.app/docs/api/rest/endpoints` and
`https://raster.app/docs/api/mcp/tools`.
