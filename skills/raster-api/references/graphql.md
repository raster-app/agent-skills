# Raster over GraphQL

One endpoint serves every operation: `POST https://api.raster.app/` with
`Authorization: Bearer <API_KEY>` and `Content-Type: application/json`. The data
model mirrors REST and MCP — same organizations, libraries, assets, and tags, and
the same permanent CDN `url` on every asset.

Resolve scope with `viewer` first; it returns `organizationId` and the
`libraries` (each `{ library, access }`) your key reaches.

## Operations

Queries

| Query          | Args                                     | Returns                     |
| -------------- | ---------------------------------------- | --------------------------- |
| `viewer`       | —                                        | `organizationId`, `libraries` |
| `libraries`    | `organizationId`                         | libraries                   |
| `assets`       | `organizationId`, `libraryId`, `tags`    | assets                      |
| `asset`        | `organizationId`, `libraryId`, `assetId` | one asset                   |
| `tags`         | `organizationId`, `libraryId`            | tags                        |
| `searchAssets` | `organizationId`, `q`, `libraries`       | `hits`                      |

Mutations

| Mutation                 | Args                                                          | Returns                   |
| ------------------------ | ------------------------------------------------------------ | ------------------------- |
| `uploadAssets`           | `organizationId`, `libraryId`, `files: [Upload!]!`, `email`  | `assets { id url }`       |
| `tagAssets`              | `organizationId`, `libraryId`, `assetIds`, `tags`            | `taggedCount`             |
| `untagAssets`            | `organizationId`, `libraryId`, `assetIds`, `tags`            | `untaggedCount`           |
| `updateAssetDescription` | `organizationId`, `libraryId`, `assetId`, `description`      | `{ assetId, description }` |
| `deleteAssets`           | `organizationId`, `libraryId`, `assets`                      | idempotent soft-delete    |
| `transferAssets`         | `organizationId`, `sourceLibraryId`, `targetLibraryId`, `assetIds` | `transferredCount`  |
| `createLibrary`          | `organizationId`, `name`, `slug`                             | new library               |
| `renameLibrary`          | `organizationId`, `libraryId`, `name`                        | renamed library           |

## Examples

Search:

```graphql
query {
  searchAssets(organizationId: "<orgId>", q: "golden hour") {
    hits { id name url tags }
    found
  }
}
```

Upload from files (GraphQL multipart `Upload` variables), returning the CDN link:

```graphql
mutation Upload($files: [Upload!]!) {
  uploadAssets(
    organizationId: "<orgId>"
    libraryId: "<libraryId>"
    files: $files
    email: "user@example.com"
  ) {
    assets { id url }
  }
}
```

Uploads process asynchronously — `id` and `url` return immediately; re-query
`assets` / `searchAssets` a few seconds later to see the asset indexed. `index`
and `trash` are reserved tags. A batch is all-or-nothing.

Full schema, arguments, and errors: `https://raster.app/docs/api/graphql`.
