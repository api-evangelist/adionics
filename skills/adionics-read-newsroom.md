---
name: Read the Adionics newsroom
description: Page the Adionics news and press archive from the public WordPress REST API, resolve authors, images and categories, and filter by date — without any credential.
api: openapi/adionics-posts-api-openapi.yml
operations: [listPosts, getPost, listCategories, getUser, getMediaItem]
---

# Read the Adionics newsroom

Adionics publishes no developer portal and no API documentation. The newsroom behind
`www.adionics.com` is nonetheless readable as JSON through the site's WordPress REST API, which
answers anonymously. Use it to track the company's project, partnership and pilot-plant
announcements.

## Before you start

- Base URL: `https://www.adionics.com/wp-json`
- **No authentication.** Send no key, no token, no header. Adding `context=edit` will get you a
  401 `rest_forbidden`.
- **No documented rate limit and no rate-limit headers.** `robots.txt` asks for `Crawl-delay: 10`.
  Honour it, and pull 100 items per request rather than 10 so you make fewer of them.
- The surface is unversioned by Adionics and can change without notice. Re-read
  `https://www.adionics.com/wp-json/` if a route stops answering.

## Steps

1. **Page the archive.** `listPosts` — `GET /wp/v2/posts?per_page=100&page=1`. Read
   `X-WP-Total` (228 at last capture) and `X-WP-TotalPages` from the response headers to size the
   loop; stop when you have consumed `X-WP-TotalPages` pages. Do not guess the end by looking for
   an empty array — a `page` beyond the last returns 400 `rest_invalid_param`.
2. **Keep the payload small.** Add `_fields=id,date,slug,link,title,excerpt,categories` unless you
   need the full rendered body. Content fields arrive wrapped as `{"rendered": "<html>"}`.
3. **Resolve references in one call.** Add `_embed` to inline the author record, the featured image
   and the category terms into an `_embedded` member, instead of following `author`,
   `featured_media` and `categories` with `getUser`, `getMediaItem` and `listCategories`.
4. **Filter by date for incremental pulls.** `GET /wp/v2/posts?after=2026-01-01T00:00:00&orderby=date&order=desc`.
   Store the newest `modified_gmt` you have seen and use it as the next `after`.
5. **Fetch one item.** `getPost` — `GET /wp/v2/posts/{id}`. A missing or unpublished id returns 404
   `rest_post_invalid_id`.

## Error handling

Errors are **not** RFC 9457. Expect `{"code": "...", "message": "...", "data": {"status": n}}`, and
branch on `code`, not on the message — messages are localised to French even on the English site.
See `errors/adionics-problem-types.yml`.

## What you cannot do

There is no write path. There is no lithium, brine, plant or project data of any kind on this API —
it serves the corporate website's content and nothing else.
