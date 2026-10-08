# Raster over REST

Base URL: `https://api.raster.app`. Every request carries two headers,
`Authorization: Bearer <API_KEY>` and `Api-Version: 2026-07-08`. The Bearer
credential can also be an OAuth access token.

Success responses carry a `data` field. Failures carry an `error` object with a
`code` and a `message` that is safe to show:

```json
{ "error": { "code": "API_KEY_READ_ONLY", "message": "Human-readable, safe to show." } }
```

Dispatch on the HTTP status first; read `error.code` when you need to branch on a
specific failure. Full codes: `https://raster.app/docs/api/rest/errors`.

## Scope

A key belongs to one organization and an allowlist of libraries, each at read or
write access. Every asset-level path takes an `:organizationId` + `:libraryId`
pair; a library the key can't reach returns `404` (treat it as out of scope).
Resolve your scope with `GET /me` before any asset call.

## Endpoints

`:orgId` is your organization slug; library paths add `/libraries/:libraryId`.

| Method   | Path                                                       | Purpose                                                     |
| -------- | ---------------------------------------------------------- | ---------------------------------------------------------- |
| `GET`    | `/me`                                                      | Resolve org + library scope.                               |
| `GET`    | `/organizations/:orgId/libraries`                          | List libraries.                                            |
| `POST`   | `/organizations/:orgId/libraries`                          | Create a library (`name`; optional `slug`).                |
| `PATCH`  | `/organizations/:orgId/libraries/:libraryId`               | Rename (`name`).                                           |
| `GET`    | `/organizations/:orgId/libraries/:libraryId/assets`        | List assets (filter `?tags=a,b`; paginate).                |
| `GET`    | `/organizations/:orgId/libraries/:libraryId/assets/:assetId` | Get one asset.                                           |
| `GET`    | `/organizations/:orgId/libraries/:libraryId/tags`          | List a library's tags.                                     |
| `POST`   | `/organizations/:orgId/libraries/:libraryId/assets`        | Upload (multipart; max 20 files).                          |
| `DELETE` | `/organizations/:orgId/libraries/:libraryId/assets`        | Soft-delete (`ids`; max 100).                              |
| `POST`   | `/organizations/:orgId/libraries/:libraryId/assets/tag`    | Tag (`assetIds`, `tags`).                                  |
| `POST`   | `/organizations/:orgId/libraries/:libraryId/assets/untag`  | Untag (`assetIds`, `tags`).                                |
| `PATCH`  | `/organizations/:orgId/libraries/:libraryId/assets/:assetId/description` | Set `description`.                            |
| `PATCH`  | `/organizations/:orgId/libraries/:libraryId/assets/:assetId/approval` | Set `approval` (`state`, `note`; OAuth token only). |
| `POST`   | `/organizations/:orgId/libraries/:libraryId/assets/transfer` | Move (source `:libraryId` → body `targetLibraryId`, `assetIds`). |
| `POST`   | `/organizations/:orgId/libraries/:libraryId/assets/:assetId/promote` | Make a variant the asset's default (`variantId`). |
| `GET`    | `/organizations/:orgId/search/assets?q=`                   | Search across authorized libraries.                        |
| `POST`   | `/libraries`                                               | Anonymous create, no `Authorization` (see no-account.md). |

## Recipes

Resolve scope with `GET /me`, which returns `data.organizationId` and `data.libraries` (each `{ library, access }`).
An OAuth token granted every organization returns `data.organizations` instead,
one entry per organization:

```bash
curl https://api.raster.app/me \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-07-08'
```

List a library's assets:

```bash
curl 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets?tags=sunset&page=1' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-07-08'
```

Upload local files (repeat `-F files=@`, max 20; let curl set the multipart `Content-Type`):

```bash
curl -X POST 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-07-08' \
  -F 'files=@/path/a.jpg' -F 'files=@/path/b.png'
```

Search (empty `q` returns recent assets; repeat `libraries` to narrow):

```bash
curl 'https://api.raster.app/organizations/<orgId>/search/assets?q=golden%20hour' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-07-08'
```

Each asset carries a permanent CDN `url`, the canonical link to hand back. An
upload returns `url` and `id` at once, and the `url` serves right away. The asset
appears in list and search a few seconds later, so re-list before calling an
upload failed.

Full endpoint reference: `https://raster.app/docs/api/rest/endpoints`. Auth and
key scoping: `https://raster.app/docs/api/rest/authentication`.
