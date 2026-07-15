---
name: markdown-negotiation
description: Use when an agent needs token-efficient, parser-friendly content from raster.app — fetch any marketing or docs page as Markdown instead of HTML by sending `Accept: text/markdown` (or appending `.md`), or grab the whole site or the whole API reference as one file. Use for reading pricing, blog posts, changelog, terms, or the docs without parsing HTML.
license: MIT
metadata:
  author: raster
  version: "1.0.0"
  homepage: https://raster.app/docs/api/skills
  source: https://github.com/raster-app/agent-skills
---

# Markdown content negotiation on raster.app

Every page under `https://raster.app` (marketing, pricing, blog, changelog, terms,
privacy, and the docs) is available as Markdown as well as HTML — a static,
token-efficient representation generated at build time.

## Fetch one page as Markdown

Send `Accept: text/markdown` on a normal `GET`; the server serves the
corresponding `.md` file with `Content-Type: text/markdown`:

```bash
curl -H "Accept: text/markdown" https://raster.app/pricing
curl -H "Accept: text/markdown" https://raster.app/docs/api/rest/endpoints
```

Or append `.md` to the path directly:

```bash
curl https://raster.app/pricing.md
curl https://raster.app/blog/why-raster.md
```

## Grab everything in one fetch

For whole-corpus context, prefer these single files over crawling page by page:

| File                                    | What it is                                              |
| --------------------------------------- | ------------------------------------------------------- |
| `https://raster.app/llms.txt`           | Sectioned index of every page + the developer surfaces. |
| `https://raster.app/llms-full.txt`      | Every marketing page concatenated as one Markdown file. |
| `https://raster.app/docs/llms-full.txt` | The complete API reference (REST, GraphQL, MCP, CLI, guides) as one file. |
| `https://api.raster.app/openapi.json`   | Machine-readable OpenAPI 3.1 description of the REST API. |

## Excluded from the rewrite

- `/.well-known/*` — agent discovery endpoints (return JSON).
- `/robots.txt` — served as plain text.
- Any path already ending in `.md`.

## Notes

- This skill covers **public content** only. For programmatic access to a user's
  libraries, assets, and tags, use the `raster-api` skill (MCP / REST / GraphQL
  with an API key).
