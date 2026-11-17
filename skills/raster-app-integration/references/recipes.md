# Recipes

Next.js shown; the SDK calls are the same in any Node.js 20.19+ server. Full
walkthrough: `https://raster.app/docs/guides/build-a-photo-album`.

## Server client with the API key

```ts
// lib/raster.ts — import only from server code
import { createRasterClient } from '@raster-app/sdk'

export const organizationId = process.env.RASTER_ORGANIZATION_ID!
export const libraryId = process.env.RASTER_LIBRARY_ID!

export const raster = createRasterClient({ apiKey: process.env.RASTER_API_KEY! })
```

## Page route

```ts
// app/api/album/route.ts
import { listAssetPage } from '@raster-app/sdk'
import { libraryId, organizationId, raster } from '@/lib/raster'

export async function GET(request: Request) {
	const page = Math.max(1, Number.parseInt(new URL(request.url).searchParams.get('page') ?? '', 10) || 1)
	const { assets, hasMore, total } = await listAssetPage(raster, { organizationId, libraryId, page })
	return Response.json({ assets, total, nextPage: hasMore ? page + 1 : null })
}
```

Search the same way: `searchAssets(raster, { organizationId, query, libraryIds: [libraryId], page })`,
and `hasMorePages(hits.length, SEARCH_PAGE_SIZE)`. Skip the call for a blank query.

## Grid tile

```tsx
'use client'
import { type Asset, getBlurhashPlaceholder, getThumbUrl } from '@raster-app/sdk'

function Tile({ asset }: { asset: Asset }) {
	const placeholder = getBlurhashPlaceholder(asset)
	return (
		<img
			src={getThumbUrl(asset)}
			alt={asset.name ?? ''}
			loading="lazy"
			style={placeholder ? { backgroundImage: `url(${placeholder})`, backgroundSize: 'cover' } : undefined}
		/>
	)
}
```

The helpers are pure functions of the asset, so they work on assets returned from your
server route. `getBlurhashPlaceholder` needs a `document`: it returns null on the server
and paints after hydration.

## Infinite scroll with prefetch

Fetch the next page before the user reaches the end, and warm its thumbnails:

```ts
function prefetchThumbs(assets: Asset[]) {
	for (const asset of assets) {
		const src = getThumbUrl(asset)
		if (src) new Image().src = src
	}
}
```

Watch a sentinel at the bottom of the grid with an `IntersectionObserver` and a generous
`rootMargin` (for example `'1500px'`). When it fires and there is a next page, fetch it,
append it (dedupe by `id`), and call `prefetchThumbs` on it. Keep one request in flight at a
time, and observe again after each page in case the sentinel is still in range.

Catch a failed page in your loader, keep the error, and stop paging until it clears, so the
observer doesn't request it again on every scroll.

With a browser credential, `useAssets` does the paging, dedupe and next-page prefetch, and
stops on error: call its `loadMore()` from the same observer, and render `assets ?? []`, since
`assets` is null until the first page arrives.

## Share link

```ts
import { getShareUrl } from '@raster-app/sdk'

const shareUrl = getShareUrl(asset)
if (shareUrl) await navigator.share?.({ url: shareUrl })
```

Send the URL rather than a `File` built from downloaded bytes. `getShareUrl` links the
original, which keeps its uploaded format. For one that many browsers can't open, such as
HEIC, share the largest size instead:

```ts
import { getPreviewUrl, parseSizes, pickSizeUrl, versionedUrl } from '@raster-app/sdk'

const large = pickSizeUrl(parseSizes(asset.sizes), 2560)
const sizeUrl = large ? versionedUrl(large, asset.contentVersion) : getPreviewUrl(asset)
```

Build the link from a fresh read of the asset, so `?v=` carries the current `contentVersion`.
