---
name: raster-app-integration
description: Use when building an app or website on Raster, such as a photo album, gallery, media grid, or CMS field that lists, renders, searches, or shares a library's assets. Covers `@raster-app/sdk` with the API key kept server-side, paging and counting a library, thumbnails with blurhash placeholders, prefetching, the processing delay for `sizes`, versioned CDN share links, and natural-language search. Use `raster-api` instead for one-off storage chores (upload, tag, hand back a URL).
license: MIT
metadata:
  author: raster
  version: "1.0.0"
  homepage: https://raster.app/docs/api/skills
  source: https://github.com/raster-app/agent-skills
---

# Building an app on Raster

Use `@raster-app/sdk` (React: `@raster-app/react`, which re-exports it). Don't
hand-roll REST calls, size parsing, or URL versioning; the SDK already does them.
Code for each step: [`references/recipes.md`](references/recipes.md).

## Credentials

- An app showing its own library: keep the API key on the server. Create the
  client in server code with `createRasterClient({ apiKey })` and return assets
  to the browser from your routes. Never ship the key to the
  browser or put it in a `NEXT_PUBLIC_` variable.
- An app where each user browses their own Raster: sign them in inside
  `<RasterProvider>` (`useSignIn`, or `signInWithDevice`), then use the hooks
  (`useAssets`, `useAssetSearch`). The hooks call Raster from the browser, so
  they need a browser credential: that sign-in, or a key every visitor may read
  connected with `connectSharedApiKey` (give it Read access to only the
  libraries they need).

## Page a library

- `listAssetPage(raster, { organizationId, libraryId, page, pageSize })` returns
  `{ assets, hasMore, total }` for one page (1-based, default 50, max 500,
  newest first). `total` is null while Raster doesn't report it; until it does,
  `hasMore` means the page came back full, so a library whose size is an exact
  multiple of `pageSize` ends with one empty page.
- `iterateAssets(raster, { organizationId, libraryId })` walks every page in
  turn, for server work such as an export.
- Never walk a whole library to count it. `getLibrary(raster, { organizationId,
  libraryId })` returns its `assetsCount`. Show what is loaded and load more on
  scroll.
- An upload during paging shifts later pages; dedupe by `id`.
- With a browser sign-in, `useAssets({ organizationId, libraryId })` pages for you:
  render `assets ?? []` (it is null until the first page arrives), call
  `loadMore()` while `hasMore`. It fetches the next page and its thumbnails
  ahead of time and returns `total`. A failed page sets `error` and stops
  paging.

## Render

- Grid tiles: `getThumbUrl(asset)`. Opened or large view: `getPreviewUrl(asset)`.
  Paint `getBlurhashPlaceholder(asset)` behind either while it loads. Never put
  the full-size `url` in a grid.
- `sizes` is a JSON string of a sparse map: only sizes smaller than the source
  exist, an SVG or small image has none, and it fills in after processing,
  seconds after upload. Read it with the SDK helpers; never hard-code its keys.
  The URL helpers pick the nearest size and fall back to `url` until sizes exist.
- Return the uploaded asset from the upload route at once and add it to the
  grid yourself: a new asset reaches lists and search a few seconds after the
  upload. Show `url` until it is processed, then re-render with a fresh read
  (`getAsset`) from a separate request. `waitUntilProcessed(raster, {
  organizationId, libraryId, assetId })` polls for about 25 seconds, then
  resolves null; keep it out of the upload request, which serverless hosts
  often cut off sooner.
- Prefetch: when the grid's sentinel nears the viewport, fetch the next page and
  preload its `getThumbUrl` images, so they are loaded before the user reaches
  them.

## Share

- Send a CDN URL rather than downloaded bytes. `getShareUrl(asset)` is the
  original file with `?v=<contentVersion>`. The original keeps its uploaded
  format; for one many browsers can't open (HEIC, say), share the largest
  size instead:
  `versionedUrl(pickSizeUrl(parseSizes(asset.sizes), 2560), asset.contentVersion)`
  when `pickSizeUrl` returns a URL, else `getPreviewUrl(asset)`.
- `contentVersion` changes when the bytes behind the same URL are replaced (a
  promoted variant, an edit), so `?v=` keeps caches from serving stale pixels.
  Build share links from a fresh read of the asset.
- `appUrl` opens the asset in Raster for a person; it is not an embed URL.

## Search

- `searchAssets(raster, { organizationId, query, libraryIds, page })` returns
  `{ hits, found, page }` (50 hits a page). Pass the user's words as typed.
- Search matches the query's words in `name`, `description`, and `tags`. On
  paid plans it also matches by meaning, so natural-language queries such as
  "kids blowing out birthday candles" work.
- A blank query returns recent assets, not matches; skip the call. A library id
  the credential can't reach fails the whole call.
- Render hits with the same helpers. A hit's `variants` is `null`; read the
  asset with `getAsset` for them.

## Common mistakes

| Mistake                                                  | Do this instead                                                          |
| -------------------------------------------------------- | ------------------------------------------------------------------------ |
| Hand-rolling REST calls and URL logic                    | Use `@raster-app/sdk`; keep the API key in a server route.               |
| Walking the whole library to count it                    | Read `assetsCount` from `getLibrary`; show what's loaded.                |
| Hard-coding `sizes` keys like `"400"` or `"1080"`        | Use `getThumbUrl` / `getPreviewUrl`; the map is sparse.                  |
| `JSON.parse(asset.sizes)` and assuming it's filled       | Sizes fill in after processing; the helpers fall back to `url`.          |
| Ignoring `contentVersion`                                | Use the helpers or `versionedUrl`, which append `?v=`.                   |
| Sharing downloaded bytes                                 | Share `getShareUrl(asset)`, or a size URL for a HEIC original.           |
| Keyword-only search UX, or filtering client-side         | Send natural-language queries to `searchAssets`; it's semantic on paid plans. |
| Full-size `url` in a grid, blank tiles while loading      | `getThumbUrl` plus `getBlurhashPlaceholder`.                             |

## Resources

- SDK: `https://www.npmjs.com/package/@raster-app/sdk`
- Guide: `https://raster.app/docs/guides/build-a-photo-album`
- SDK reference: `https://raster.app/docs/api/sdk`
- Asset fields: `https://raster.app/docs/api/objects`
- Storage chores over MCP or REST: the `raster-api` skill.
