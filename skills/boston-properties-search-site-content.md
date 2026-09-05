---
name: boston-properties-search-site-content
description: Search and retrieve BXP (Boston Properties) corporate website content through the WordPress REST API the company serves at www.bxp.com/wp-json. Use when you need the text of a BXP page, a content search across bxp.com, or the site's content taxonomy. Do NOT use it to look for property, leasing or availability data — BXP does not expose those.
api: BXP WordPress REST API
base_url: https://www.bxp.com/wp-json
operations:
  - getWpV2Search
  - getWpV2Pages
  - getWpV2PagesById
  - getWpV2Categories
  - getWpV2Types
generated: '2026-09-04'
method: generated
source: openapi/boston-properties-wordpress-rest-openapi.yml
---

# Search BXP site content

BXP runs no developer program and publishes no API documentation. This skill operates the one
contract the company actually serves: the WordPress REST API on its corporate site. Reads are
anonymous; nothing here requires a key, and no key is obtainable.

## Before you start

**Send a browser-shaped `User-Agent`.** Cloudflare answers a default agent User-Agent with an
HTML 403 interstitial (`Attention Required! | Cloudflare`) on *every* route, including the ones
that return 200 to a browser. If you get HTML back, this is why — it is not a rate limit and
retrying with the same header will not help.

Respect `Crawl-delay: 10` from https://www.bxp.com/robots.txt.

## Steps

1. **Search.** `getWpV2Search` — `GET /wp/v2/search?search=<terms>&per_page=20`.
   Returns lightweight hits: `id`, `title`, `url`, `type`, `subtype`, plus `_links.self` pointing
   at the full record. Use `subtype=page` to narrow to pages.
2. **Fetch the record.** Follow `_links.self`, or call `getWpV2PagesById` —
   `GET /wp/v2/pages/{id}`. The body text is in `content.rendered` as HTML; the summary is in
   `excerpt.rendered`.
3. **Browse instead of searching** when you want everything. `getWpV2Pages` —
   `GET /wp/v2/pages?per_page=100&orderby=modified&order=desc`. Twenty pages were visible
   anonymously on 2026-09-04.
4. **Narrow the payload.** Every route accepts `_fields` (a sparse fieldset) and `_embed`.
   `GET /wp/v2/pages?_fields=id,slug,title,link,modified` is dramatically smaller than the
   default response, which includes a large `yoast_head` SEO blob on every record.
5. **Understand the shape first, if unsure.** `getWpV2Types` — `GET /wp/v2/types` — lists exactly
   which content types this site exposes. `getWpV2Categories` — `GET /wp/v2/categories` — lists
   the taxonomy with post counts.

## Pagination

`page` (from 1) and `per_page` (max **100**; anything larger is a 400 `rest_invalid_param`).
Read `X-WP-Total` and `X-WP-TotalPages` from the response headers, or follow the RFC 8288
`Link: <...>; rel="next"` header. Do not guess the number of pages.

## Errors

The envelope is WordPress's, not RFC 9457: `{"code": ..., "message": ..., "data": {"status": ...}}`.

| code | status | what it means |
|---|---|---|
| `rest_invalid_param` | 400 | a parameter failed validation; read `data.params` |
| `rest_no_route` | 404 | that path/method pair does not exist |
| `rest_post_invalid_id` | 404 | no record with that id |
| `rest_forbidden` | 401 | the route needs a BXP WordPress account — stop, you cannot get one |
| `rest_cannot_view_themes` | 401 | capability-specific refusal, same conclusion |

A 403 with an HTML body is Cloudflare, not the API. See errors/boston-properties-problem-types.yml.

## What this API will not give you

Properties, regions, team members, property types and member types exist on bxp.com — each has
its own XML sitemap — but **none of them is registered for REST**. There is no leasing,
availability, tenant or building-systems API. If the task needs that data, the answer is that
BXP does not publish it programmatically; scrape the HTML at https://www.bxp.com/properties or
use the contact form. See data-model/boston-properties-data-model.yml.

## Do not write

Every mutating route requires a WordPress account on BXP's own site. There is no idempotency
mechanism and no dry-run mode. Do not attempt writes against a production corporate site.
