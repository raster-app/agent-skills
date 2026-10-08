# Searching and listing assets

Three read verbs find assets and return the permanent CDN `url` you hand back.

## search_assets — across libraries

`search_assets` (REST `GET /organizations/:orgId/search/assets?q=`) runs a full-text query across every library your credential can reach,
matching `name`, `description`, and `tags`. Hits come back ranked by relevance,
with highlighted snippets.

- Empty or whitespace `q` returns recent assets in scope (no ranking), a quick
  look at what's in there.
- Pass `libraries` to narrow to a subset (REST: repeat the param, or
  comma-separate it as `?libraries=a,b`). Naming a library the key can't reach
  fails the whole call.
- Pass `approval` (`in_review`, `approved`, `needs_changes`, or `none`) to return
  only assets in that approval state.

```bash
curl 'https://api.raster.app/organizations/<orgId>/search/assets?q=golden%20hour&libraries=<libraryId>' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-07-08'
```

## list_assets — one library

`list_assets` (REST `GET .../libraries/:libraryId/assets`) pages
through a single library. Filter with `tags` (max 50) to return only assets
carrying any of them; paginate with `page` / `pageSize`.

## get_asset — one by id

`get_asset` (REST `GET .../assets/:assetId`) returns a single asset
by the `id` from a list, search, or upload.

## What to return

Each asset carries a permanent CDN `url`, the canonical asset link. Hand that
`url` to the user. Pass its `tags` and `description` back as context, and point
to the library at `raster.app/<organizationId>/<libraryId>`.

A new upload appears in search and lists a few seconds after the upload returns,
so re-run the query before concluding an asset is missing.

Full reference: `https://raster.app/docs/api/rest/endpoints` and
`https://raster.app/docs/api/mcp/tools`.
