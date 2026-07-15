# Raster over MCP

Endpoint: `https://mcp.raster.app/` (Streamable HTTP). Point any MCP-capable
client at it — Claude, Cursor, VS Code, the OpenAI Responses API.

Register it as a remote MCP server with an `Authorization: Bearer <API_KEY>`
header, then call `whoami` first: it returns the `organizationId` and the
libraries your key reaches. Every asset call needs that `organizationId` + a
`libraryId`; `search_assets` is organization-scoped with an optional `libraries`
subset. A library the key can't reach returns an out-of-scope (404) error.
`tools/list` returns each tool's authoritative input schema — read it for exact
argument names and types.

## Tools

Scope

| Tool     | Purpose                                             |
| -------- | -------------------------------------------------- |
| `whoami` | Resolve the key's organization + library scope. Call first. |

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
| `transfer_assets`          | Move assets between libraries in the org.        |
| `delete_assets`            | Soft-delete up to 100 (recoverable from trash).  |

Structure

| Tool             | Purpose                                       |
| ---------------- | -------------------------------------------- |
| `create_library` | New library in an org your key can write.    |
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

- Uploads are asynchronous — `url`/`id` return at once; the asset appears in `list_assets` / `search_assets` seconds later, so re-list before concluding failure.
- `index` and `trash` are reserved tags — `tag_assets` rejects them with `BAD_USER_INPUT`.
- Batches are all-or-nothing: one bad source or out-of-scope id fails the whole `upload_assets` / `tag_assets` / `transfer_assets` call.
- The `apiKey` from `create_organization` is shown once — persist it immediately.

Overview: `https://raster.app/docs/api/mcp`. Every tool, argument, and example:
`https://raster.app/docs/api/mcp/tools`.
