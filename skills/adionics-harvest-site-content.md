---
name: Harvest the Adionics site content and media
description: Pull the full static page tree (English and Spanish), the media library and the classification terms from the Adionics WordPress REST API for indexing or corpus building.
api: openapi/adionics-pages-api-openapi.yml
operations: [listPages, getPage, listMedia, getMediaItem, listCategories, listTaxonomies, listLanguages, search, getRouteIndex]
---

# Harvest the Adionics site content and media

Use this to build a complete, machine-readable copy of what Adionics publishes about its Flionex
direct-lithium-extraction process — the technology, offer, markets, applications, history, team and
careers pages — plus the images, diagrams and brochures attached to them.

## Before you start

- Base URL: `https://www.adionics.com/wp-json`. No credential.
- The site is **bilingual**. `listLanguages` (`GET /pll/v1/languages`) returns English (`en_GB`) and
  Spanish (`es_ES`); the Spanish tree lives under `/es/`. Both trees are returned together by
  `listPages`, so de-duplicate on `link` prefix if you want one language.

## Steps

1. **Confirm the surface.** `getRouteIndex` — `GET /wp-json/`. It is self-describing; if a route in
   this skill is gone, it will not be listed.
2. **Pull every page.** `listPages` — `GET /wp/v2/pages?per_page=100&_fields=id,slug,link,parent,title,content,modified`.
   29 pages at last capture, so one request covers it. `parent` gives you the hierarchy
   (`/in-few-words/careers/` is a child of `/in-few-words/`).
3. **Pull the media library.** `listMedia` — `GET /wp/v2/media?per_page=100&_fields=id,title,mime_type,source_url,alt_text,media_details,post`.
   551 attachments; page with `X-WP-TotalPages`. `source_url` is the direct file; `media_details.sizes`
   holds the generated renditions; `post` links the attachment back to the page or post that uses it.
   Filter brochures with `mime_type=application/pdf`.
4. **Pull the classification.** `listCategories` (4 terms) and `listTaxonomies`. The tag vocabulary
   is registered but empty — `listTags` returns `X-WP-Total: 0`, which is a real answer, not a fault.
5. **Cross-check with search.** `search` — `GET /wp/v2/search?per_page=100` returns 267 searchable
   objects (posts plus pages) as lightweight `id`/`title`/`url`/`subtype` records. Use it to confirm
   you missed nothing.

## Conventions that matter

- Sparse fieldsets via `_fields`; embedded relations via `_embed`. Both are described in
  `conventions/adionics-conventions.yml`.
- Rendered HTML arrives inside `{"rendered": ...}` and contains theme markup — strip tags before
  indexing.
- Honour `Crawl-delay: 10` from `robots.txt`; there is no published rate limit and no runtime signal.

## Do not

Do not attempt `context=edit`, `/wp/v2/settings`, `/wp/v2/plugins`, `/wp/v2/themes`,
`/wp-abilities/v1/*` or the MCP endpoint — all return 401 `rest_forbidden` without a WordPress
Application Password, which Adionics does not issue to the public.
