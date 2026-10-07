# Organizing assets

Make assets findable and move them between libraries. Each operation is scoped to
one library (`organizationId` + `libraryId`) and, except `update_asset_description`,
batched.

## Tag and untag

`tag_assets` / `untag_assets` (REST `POST .../assets/tag` and `/untag`) take `assetIds` and `tags` and return `taggedCount` /
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

`update_asset_description` (REST `PATCH .../assets/:assetId/description`) replaces one asset's `description`, stored verbatim — no
trim, no rewrite. Pass an empty string to clear it. It echoes back
`{ assetId, description }`.

## Set approval

`set_asset_approval` (REST `PATCH .../assets/:assetId/approval`) sets one asset's
`approval` to `in_review`, `approved`, or `needs_changes`, with an optional `note`
(up to 500 characters). Each call replaces the previous approval. It echoes back
`{ assetId, approval }`.

- It needs an **OAuth** connection or access token. An API key reads `approval` but
  setting it fails with `403 APPROVAL_REQUIRES_USER` — ask the user to connect over
  OAuth instead of retrying.
- Setting approval needs a paid plan: without one it fails with
  `403 PAID_PLAN_REQUIRED`. Surface it to the user; don't retry.
- `approved` and `needs_changes` email the asset's uploader.

## Transfer between libraries

`transfer_assets` (REST `POST .../assets/transfer`) moves
assets within the same organization. The source is the `:libraryId` in the path
(MCP `sourceLibraryId`); the body names `targetLibraryId` and `assetIds`.
Every id must currently live in the source library, or the whole call fails. Returns
`transferredCount`.

## Delete

`delete_assets` (REST `DELETE .../assets` with body `ids`) is a **soft delete** — up to 100 ids move to trash, leave lists and
search at once, and stay recoverable. It is idempotent: re-deleting a trashed id is
a no-op.

Full reference: `https://raster.app/docs/api/rest/endpoints` and
`https://raster.app/docs/api/mcp/tools`. Task guide:
`https://raster.app/docs/guides/organizing-images`.
