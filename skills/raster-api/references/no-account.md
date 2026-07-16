# Starting with no account

An agent holding no key can mint a real organization, a starter library, and a
usable API key from a user's email — in one call, sent with no `Authorization`
header.

- **MCP** — connect to the no-credential endpoint `https://mcp.raster.app/anonymous` and call `create_organization` with `{ "email": "user@example.com", "name": "Q3 campaign" }`; then connect to `https://mcp.raster.app/` with the returned key for everything else.
- **REST** — `POST https://api.raster.app/libraries` with the `Api-Version: 2026-05-20`
  header and a JSON body `{ "email": "…", "name": "…" }`. The endpoint is anonymous —
  it mints the key, so send no `Authorization` header.

Both return `organizationId`, `libraryId`, `apiKey`, and `claimUrl`.

```bash
curl -X POST 'https://api.raster.app/libraries' \
  -H 'Api-Version: 2026-05-20' -H 'Content-Type: application/json' \
  -d '{"email":"user@example.com","name":"Q3 campaign"}'
```

## After the call

1. **Persist the `apiKey` immediately** — it is shown once and stored only encrypted
   afterward. Losing it means losing access until the user claims the library.
2. **Use it as your bearer token** for every later call: `Authorization: Bearer <apiKey>`
   (plus `Api-Version` over REST). The returned `organizationId` + `libraryId` are the
   target for upload, tag, search, and the rest of the surface.
3. **Hand the user the `claimUrl`.** They open it, sign in, and the library and its
   assets fold into a real account; the key becomes one they manage in settings.
   Surface the `claimUrl` yourself so the hand-off stands on its own.

An unclaimed library is held for a limited window and removed if the window closes.
Creating past the per-email cap on active unclaimed libraries returns
`400 BAD_USER_INPUT`.

Full guide: `https://raster.app/docs/guides/start-without-an-account`.
