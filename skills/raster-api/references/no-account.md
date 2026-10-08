# Starting with no account

An agent holding no key can mint a real organization, a starter library, and a
usable API key from a user's email in one call, sent with no `Authorization`
header.

- Over MCP, connect to the no-credential endpoint `https://mcp.raster.app/anonymous` and call `create_organization` with `{ "email": "<your-email>", "name": "Q3 campaign" }`. Then connect to `https://mcp.raster.app/` with the returned key for everything else.
- Over REST, send `POST https://api.raster.app/libraries` with the `Api-Version: 2026-07-08`
  header and a JSON body `{ "email": "<your-email>", "name": "Q3 campaign" }`. The endpoint is anonymous:
  it mints the key, so send no `Authorization` header.

Both return `organizationId`, `libraryId`, `apiKey`, and `claimUrl`.

```bash
curl -X POST 'https://api.raster.app/libraries' \
  -H 'Api-Version: 2026-07-08' -H 'Content-Type: application/json' \
  -d '{"email":"<your-email>","name":"Q3 campaign"}'
```

## After the call

1. Persist the `apiKey` immediately. It is shown once, and losing it means
   losing access until the user claims the library.
2. Use it as your bearer token for every later call: `Authorization: Bearer <apiKey>`
   (plus `Api-Version` over REST). The returned `organizationId` + `libraryId` are the
   target for upload, tag, search, and the rest of the surface.
3. Hand the user the `claimUrl`. They open it and sign in, and the library and its
   assets move into a real account; the key becomes one they manage in settings.
   Surface the `claimUrl` yourself so the hand-off does not depend on email delivery.

An unclaimed library is held for a limited window and removed if the window closes.
Creating past the per-email cap on active unclaimed libraries returns
`400 BAD_USER_INPUT`.

Full guide: `https://raster.app/docs/guides/start-without-an-account`.
