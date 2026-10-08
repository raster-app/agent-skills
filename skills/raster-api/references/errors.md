# Raster error reference

REST and MCP return identical HTTP statuses and messages for the same coded
failure. Over MCP, arguments that fail a tool's input schema are refused before
the tool runs, with a plain-text `-32602` error that has no `code` or `status`.
Every REST failure body is shaped:

```json
{ "error": { "code": "ERROR_CODE", "message": "Human-readable, safe to show." } }
```

Dispatch on the HTTP status first (no body parsing needed), and use `code` when
you need to react to a specific failure. Canonical:
`https://raster.app/docs/api/rest/errors`.

## Status mapping

| Status | When                                                                          |
| ------ | ----------------------------------------------------------------------------- |
| 400    | Malformed request: missing `Api-Version`, bad JSON, invalid params, too many tags. Also returned when an org is at its library limit. |
| 401    | Missing, malformed (non-`Bearer`), or unknown `Authorization` header, or a rejected OAuth access token. |
| 403    | Authenticated but not allowed: a read-only key, a credential or role that can't perform the operation, or a plan or storage cap. |
| 404    | Unknown org, asset, or path, or a key without access to the target library.   |
| 409    | The library `slug` is already taken, or a promote is already running on the asset. |
| 413    | A REST file exceeds the per-file upload size limit, or an MCP request body is over 32 MB. |
| 500    | Unexpected server fault. Safe to retry with backoff.                          |
| 503    | An OAuth access token couldn't be verified for a moment. Safe to retry with backoff. |

## Codes

| Code                                | HTTP | Meaning / what to do                                                                 |
| ----------------------------------- | ---- | ----------------------------------------------------------------------------------- |
| `UNAUTHENTICATED`                   | 401  | `Authorization` missing or not `Bearer <credential>`, or an OAuth access token that is invalid, expired, or revoked. Send the header, or re-authorize the connection. |
| `INVALID_API_KEY`                   | 401  | The bearer token isn't a key in that organization.                                  |
| `API_VERSION_REQUIRED`              | 400  | Add `Api-Version: 2026-07-08`.                                                       |
| `API_KEY_NOT_AUTHORIZED_FOR_LIBRARY` | 404 | Key can't reach that library: it doesn't exist or isn't granted, and both get the same 404. Pick one `whoami` lists instead of retrying. |
| `API_KEY_READ_ONLY`                 | 403  | A read-only key (no Write on any library) tried to create a library. Use a key with Write access. |
| `FORBIDDEN`                         | 403  | An OAuth connection tried to create a library and the user isn't an Owner of the organization, or setting approval isn't available to the user's account. |
| `APPROVAL_REQUIRES_USER`            | 403  | An API key tried to set approval. Setting it needs an OAuth connection; ask the user to connect. |
| `PAID_PLAN_REQUIRED`                | 403  | Setting approval needs a paid plan and the org has none. The user must upgrade. |
| `ORGANIZATION_NOT_FOUND`            | 404  | `:organizationId` doesn't resolve to a known org.                                   |
| `RESOURCE_NOT_FOUND`                | 404  | The referenced asset doesn't exist.                                                 |
| `NOT_FOUND`                         | 404  | The variant a promote names isn't in the library, or is in the trash.               |
| `ENDPOINT_NOT_FOUND`                | 404  | Unknown REST path (distinct from a missing resource).                               |
| `BAD_USER_INPUT`                    | 400  | Invalid body or params: malformed JSON, a reserved `index`/`trash` tag, a bad email, an exhausted per-email create quota, more than 20 files in an upload, a delete with no `ids` or more than 100, or an upload with no file or no accepted file (the message names each file and why). Fix and resend. |
| `PAYLOAD_TOO_LARGE`                 | 413  | A REST file exceeds the per-file size limit, or an MCP request body is over 32 MB. Shrink or split. |
| `LIBRARY_URL_TAKEN`                 | 409  | The `slug` for a new library is already taken in the organization. Choose a different slug. |
| `CONFLICT`                          | 409  | A promote is already running on the asset. Wait for it to finish, then retry.       |
| `PRO_PLAN_CANCELED`                 | 403  | The org's Pro plan ended and it still has more than one member. The user must reactivate it, or remove all but one member. Returned for uploads and library creation. |
| `STORAGE_LIMIT_EXCEEDED`            | 403  | Free-plan 1 GB storage cap reached. The user must upgrade.                          |
| `LIBRARY_LOCKED`                    | 403  | Free plan includes only the first 3 libraries; this one is locked.                  |
| `EPHEMERAL_STORAGE_LIMIT_EXCEEDED`  | 403  | An unclaimed temporary library hit its cap. The user must claim it to keep uploading. |
| `LIBRARY_LIMIT_EXCEEDED`            | 400  | A Free-plan org already has 3 or more libraries and tried to create another. The user must upgrade to Pro. |
| `EPHEMERAL_LIBRARY_LIMIT_EXCEEDED`  | 400  | A temporary (unclaimed) library can't create more libraries. Claim it first.        |
| `INTERNAL_SERVER_ERROR`             | 500  | Server fault. Retry with backoff.                                                   |
| `SERVICE_UNAVAILABLE`               | 503  | An OAuth access token couldn't be verified for a moment. Retry with the same token, with backoff. |

## Handling pattern

- On a 401, fix the key or header, or re-authorize the connection. Don't retry
  the same request.
- On a 404 from an asset call, the library or asset is out of scope. Re-resolve
  with `whoami` / `list_libraries` and pick a reachable one.
- On a 400 `BAD_USER_INPUT`, the input is wrong and the `message` says how.
  Don't retry it unchanged.
- A 403 for a plan or storage cap needs a human decision (upgrade, renew, or
  claim). Surface the message to the user.
- A 500 or 503 is transient. Retry with exponential backoff.
