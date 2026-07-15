# Organizing assets

Make assets findable and move them between libraries. Each operation is scoped to
one library (`organizationId` + `libraryId`) and, except `update_asset_description`,
batched.

## Tag and untag

`tag_assets` / `untag_assets` (REST `POST .../assets/tag` and `/untag`, GraphQL
`tagAssets` / `untagAssets`) take `assetIds` and `tags` and return `taggedCount` /
`untaggedCount` — the count of `(asset, tag)` pairs that actually changed. Both are
idempotent: re-tagging a pair the asset already has, or untagging one it lacks, is a
silent skip.

- `index` and `trash` are **reserved tags** — either call rejects them with
  `BAD_USER_INPUT` before any write.
- Every asset must belong to the named library; any mismatch fails the whole call.
- Up to 100 `assetIds` and 20 `tags` per call.

```bash
curl -X POST 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets/tag' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20' \
  -H 'Content-Type: application/json' \
  -d '{"assetIds":["<id1>","<id2>"],"tags":["sunset","landscape"]}'
```

## Describe

`update_asset_description` (REST `PATCH .../assets/:assetId/description`, GraphQL
`updateAssetDescription`) replaces one asset's `description`, stored verbatim — no
trim, no rewrite. Pass an empty string to clear it. It echoes back
`{ assetId, description }`.

## Transfer between libraries

`transfer_assets` (REST `POST .../assets/transfer`, GraphQL `transferAssets`) moves
assets within the same organization. The source is the `:libraryId` in the path
(MCP/GraphQL `sourceLibraryId`); the body names `targetLibraryId` and `assetIds`.
Every id must currently live in the source library, or the whole call fails. Returns
`transferredCount`.

## Delete

`delete_assets` (REST `DELETE .../assets` with body `ids`, GraphQL `deleteAssets`
with `assets`) is a **soft delete** — up to 100 ids move to trash, leave lists and
search at once, and stay recoverable. It is idempotent: re-deleting a trashed id is
a no-op.

Full reference: `https://raster.app/docs/api/rest/endpoints` and
`https://raster.app/docs/api/mcp/tools`. Task guide:
`https://raster.app/docs/guides/organizing-images`.
