# Raster error reference

REST and GraphQL return **identical** HTTP statuses and messages for the same
coded failure. Every REST failure body is shaped:

```json
{ "error": { "code": "ERROR_CODE", "message": "Human-readable, safe to show." } }
```

Dispatch on the HTTP **status** first (no body parsing needed); use `code` when
you need to react to a specific failure. Canonical:
`https://raster.app/docs/api/rest/errors`.

## Status mapping

| Status | When                                                                          |
| ------ | ----------------------------------------------------------------------------- |
| 400    | Malformed request — missing `Api-Version`, bad JSON, invalid params, too many tags. |
| 401    | Missing, malformed (non-`Bearer`), or unknown `Authorization` header.         |
| 403    | Authenticated but not allowed — read-only key, or a plan/storage cap.         |
| 404    | Unknown org / asset / path, or a key without access to the target library.    |
| 413    | A file exceeds the per-file upload size limit.                                |
| 500    | Unexpected server fault. Safe to retry with backoff.                          |

## Codes

| Code                                | HTTP | Meaning / what to do                                                                 |
| ----------------------------------- | ---- | ----------------------------------------------------------------------------------- |
| `UNAUTHENTICATED`                   | 401  | `Authorization` missing or not `Bearer <key>`. Send the header.                     |
| `INVALID_API_KEY`                   | 401  | The bearer token isn't a key in that organization.                                  |
| `API_VERSION_REQUIRED`              | 400  | Add `Api-Version: 2026-05-20`.                                                       |
| `API_KEY_NOT_AUTHORIZED_FOR_LIBRARY` | 404 | Key can't reach that library (unknown or not granted). Pick one `whoami` lists. 404 hides existence — not "retry". |
| `API_KEY_READ_ONLY`                 | 403  | A read-only key (no Write on any library) tried to create a library. Use a key with Write access. |
| `ORGANIZATION_NOT_FOUND`            | 404  | `:organizationId` doesn't resolve to a known org.                                   |
| `RESOURCE_NOT_FOUND`                | 404  | The referenced asset doesn't exist.                                                 |
| `ENDPOINT_NOT_FOUND`                | 404  | Unknown REST path (distinct from a missing resource).                               |
| `BAD_USER_INPUT`                    | 400  | Invalid body/params — malformed JSON, reserved `index`/`trash` tag, bad email, exhausted per-email create quota. Fix and resend. |
| `PAYLOAD_TOO_LARGE`                 | 413  | A file exceeds the per-file size limit. Shrink or split.                            |
| `MAX_ASSETS_EXCEEDED`               | 400  | >20 files in an upload, or >100 ids in a delete. Split the batch.                   |
| `UPLOAD_ERROR_NO_FILES`             | 400  | The upload carried no usable file parts.                                            |
| `DELETE_ERROR`                      | 400  | The delete request is missing `ids` or `ids` is empty.                              |
| `PRO_PLAN_CANCELED`                 | 403  | The org's Pro subscription is canceled. The user must renew to upload.              |
| `STORAGE_LIMIT_EXCEEDED`            | 403  | Free-plan 1 GB storage cap reached. The user must upgrade.                          |
| `LIBRARY_LOCKED`                    | 403  | Free plan includes only the first 3 libraries; this one is locked.                  |
| `EPHEMERAL_STORAGE_LIMIT_EXCEEDED`  | 403  | An unclaimed temporary library hit its cap. The user must claim it to keep uploading. |
| `EPHEMERAL_LIBRARY_LIMIT_EXCEEDED`  | 400  | A temporary (unclaimed) library can't create more libraries. Claim it first.        |
| `INTERNAL_SERVER_ERROR`             | 500  | Server fault. Retry with backoff.                                                   |

## Handling pattern

- **401** → fix the key/header; don't retry the same request.
- **404 on an asset call** → the library or asset is out of scope; re-resolve with `whoami` / `list_libraries` and pick a reachable one.
- **400 `BAD_USER_INPUT`** → the input is wrong; the `message` says how. Don't retry unchanged.
- **403 plan/storage** → a human decision (upgrade/renew/claim); surface the message to the user.
- **500** → transient; retry with exponential backoff.
