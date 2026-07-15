# Raster over REST

Base URL: `https://api.raster.app`. Every request carries two headers —
`Authorization: Bearer <API_KEY>` and `Api-Version: 2026-05-20`.

Success responses carry a `data` field; failures carry an `error` object with a
`code` and a human-safe `message`:

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
| `POST`   | `/organizations/:orgId/libraries/:libraryId/assets`        | Upload (multipart; max 20 files).                          |
| `DELETE` | `/organizations/:orgId/libraries/:libraryId/assets`        | Soft-delete (`ids`; max 100).                              |
| `POST`   | `/organizations/:orgId/libraries/:libraryId/assets/tag`    | Tag (`assetIds`, `tags`).                                  |
| `POST`   | `/organizations/:orgId/libraries/:libraryId/assets/untag`  | Untag (`assetIds`, `tags`).                                |
| `PATCH`  | `/organizations/:orgId/libraries/:libraryId/assets/:assetId/description` | Set `description`.                            |
| `POST`   | `/organizations/:orgId/libraries/:libraryId/assets/transfer` | Move (source `:libraryId` → body `targetLibraryId`, `assetIds`). |
| `GET`    | `/organizations/:orgId/search/assets?q=`                   | Search across authorized libraries.                        |
| `POST`   | `/libraries`                                               | Anonymous create — no `Authorization` (see no-account.md). |

## Recipes

Resolve scope — returns `data.organizationId` and `data.libraries` (each `{ library, access }`):

```bash
curl https://api.raster.app/me \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20'
```

List a library's assets:

```bash
curl 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets?tags=sunset&page=1' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20'
```

Upload local files (repeat `-F files=@`, max 20; let curl set the multipart `Content-Type`):

```bash
curl -X POST 'https://api.raster.app/organizations/<orgId>/libraries/<libraryId>/assets' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20' \
  -F 'files=@/path/a.jpg' -F 'files=@/path/b.png'
```

Search (empty `q` returns recent assets; repeat `libraries` to narrow):

```bash
curl 'https://api.raster.app/organizations/<orgId>/search/assets?q=golden%20hour' \
  -H 'Authorization: Bearer <API_KEY>' -H 'Api-Version: 2026-05-20'
```

Each asset carries a permanent CDN `url` — the canonical link to hand back.
Uploads process asynchronously: `url` and `id` return at once, but the asset
lands in list and search a few seconds later — re-list before calling one failed.

Full endpoint reference: `https://raster.app/docs/api/rest/endpoints`. Auth and
key scoping: `https://raster.app/docs/api/rest/authentication`.
