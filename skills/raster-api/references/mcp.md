# Raster over MCP

Endpoint: `https://mcp.raster.app/` (Streamable HTTP), for any MCP-capable
client, such as Claude, Cursor, VS Code, or the OpenAI Responses API.

## Connect

OAuth is recommended. Add `https://mcp.raster.app/` as a remote MCP server. The
client discovers the authorization server, registers itself, and opens a Raster
consent page where you sign in and choose the organizations the connection may
access. When the client requests access to every organization, you grant all of
yours (the default) or narrow to one; otherwise you pick one. With every
organization granted, `whoami` returns an `organizations` list instead of a
single `organizationId`. The connection uses your library access and follows it
live. Revoke it under Settings → Connected apps.

For server-to-server use, paste an organization-scoped API key as an
`Authorization: Bearer <API_KEY>` header when adding the server.

With no account yet, connect to `https://mcp.raster.app/anonymous` (no
credential). It serves only `create_organization`, which mints an organization, a
starter library, and an API key. Then connect to `https://mcp.raster.app/` with
that key. See no-account.md.

## Working

Call `whoami` first. It returns the `organizationId` and the libraries you reach.
Every asset call needs that `organizationId` and a `libraryId`; `search_assets` is
organization-scoped with an optional `libraries` subset. A library you can't reach
returns an out-of-scope (404) error. `tools/list` returns each tool's authoritative
input schema, so read it for exact argument names and types.

## Tools

Scope

| Tool     | Purpose                                             |
| -------- | -------------------------------------------------- |
| `whoami` | Resolve your credential's organization + library scope. Call first. |

Read

| Tool             | Purpose                                                |
| ---------------- | ----------------------------------------------------- |
| `list_libraries` | Libraries in an organization.                         |
| `list_assets`    | Assets in one library (paginate; `tags` filter).      |
| `search_assets`  | Ranked search across authorized libraries (highlights). |
| `list_tags`      | A library's tags by usage.                            |
| `get_asset`      | One asset by id.                                      |

Write

| Tool                       | Purpose                                          |
| -------------------------- | ------------------------------------------------ |
| `upload_asset`             | Upload one file (`source`: URL or base64).       |
| `upload_assets`            | Upload up to 20 files (`sources[]`).             |
| `tag_assets`               | Apply tags (`assetIds`, `tags`).                 |
| `untag_assets`             | Remove tags.                                     |
| `update_asset_description` | Replace one asset's description (verbatim).      |
| `set_asset_approval`       | Set one asset's approval state (OAuth only).     |
| `transfer_assets`          | Move assets between libraries in the org.        |
| `delete_assets`            | Soft-delete up to 100 (recoverable from trash for 30 days). |
| `promote_variant`          | Make a variant the asset's default, at the asset's existing URL. |

Structure

| Tool             | Purpose                                       |
| ---------------- | -------------------------------------------- |
| `create_library` | New library in an org (needs a write key, or the owner role over OAuth). |
| `rename_library` | Rename a library.                            |

No-account

| Tool                  | Purpose                                                     |
| --------------------- | ---------------------------------------------------------- |
| `create_organization` | Mint an org + library + API key from an email (see no-account.md). |

## Example call

A `tools/call` names the tool and an `arguments` object (`whoami` takes `{}`):

```json
{
  "name": "upload_assets",
  "arguments": {
    "organizationId": "<orgId>",
    "libraryId": "<libraryId>",
    "sources": [{ "kind": "url", "url": "https://example.com/photo.jpg" }]
  }
}
```

## Gotchas

- An upload returns `url` and `id` at once, and the `url` serves right away. The asset appears in `list_assets` / `search_assets` a few seconds later, so re-list before concluding an upload failed.
- `index` and `trash` are reserved tags. `tag_assets` rejects them with `BAD_USER_INPUT`.
- One out-of-scope id fails the whole `upload_assets` / `tag_assets` / `transfer_assets` call. An upload also fails as a whole when a source fails validation (its schema, or a URL that resolves to a private network), a URL can't be fetched, or the request body is over 32 MB.
- A file refused for its type, one that can't be opened, or a URL file over 1 GB is left out and named in `responseText` while the rest upload.
- The `apiKey` from `create_organization` is shown once. Persist it immediately.

Overview: `https://raster.app/docs/api/mcp`. Every tool, argument, and example:
`https://raster.app/docs/api/mcp/tools`.
