---
name: boston-properties-monitor-site-changes
description: Track what changed on the BXP (Boston Properties) corporate site — new or edited pages, new news items — using the WordPress REST API and the site's sitemaps and RSS feed. Use for competitive monitoring, press tracking or checking whether a BXP page has been updated since a given date.
api: BXP WordPress REST API
base_url: https://www.bxp.com/wp-json
operations:
  - getWpV2Pages
  - getWpV2Posts
  - getWpV2Media
  - getWpV2Categories
generated: '2026-09-04'
method: generated
source: openapi/boston-properties-wordpress-rest-openapi.yml
---

# Monitor BXP site changes

## Before you start

Send a browser-shaped `User-Agent` — Cloudflare returns an HTML 403 interstitial to default agent
User-Agents on every route. Honour `Crawl-delay: 10`. Responses are edge-cached with
`cache-control: max-age=600`, so polling faster than ten minutes buys you nothing.

## Steps

1. **Pages changed since a date.** `getWpV2Pages` —
   `GET /wp/v2/pages?modified_after=2026-08-01T00:00:00&orderby=modified&order=desc&_fields=id,slug,link,title,modified`.
   `modified_after` / `modified_before` and `after` / `before` (on `date`) are real parameters on
   this route; they are in BXP's own route index.
2. **Read the count, not the array length.** `X-WP-Total` and `X-WP-TotalPages` tell you the true
   size of the result; page with `page` and `per_page` (max 100) or follow `Link: rel="next"`.
3. **Posts.** `getWpV2Posts` — `GET /wp/v2/posts?after=<iso>`. Note that on 2026-09-04 this
   returned an empty array: BXP's news archive is **not** stored as WordPress core posts.
   For news, use the RSS feed at https://www.bxp.com/news/feed/ or the news sitemap at
   https://www.bxp.com/news-sitemap.xml (126 entries) instead of this operation.
4. **Media.** `getWpV2Media` — `GET /wp/v2/media?after=<iso>` for newly uploaded assets
   (reports, images). Returned empty anonymously on 2026-09-04.
5. **Sitemaps are the fuller signal.** The REST API exposes only WordPress core types. The
   sitemaps at https://www.bxp.com/sitemap_index.xml cover property, region, team-member,
   property-type and member-type content that the API does not, and each `<loc>` carries a
   `lastmod`. Diff the sitemaps to detect portfolio and personnel changes.

## Diffing safely

Record `id` + `modified` per page and compare across runs. `id` is a site-local WordPress
integer — stable for a given record on this site, but not a public BXP identifier and not
meaningful anywhere else. Prefer `slug` or `link` when persisting.

## Errors and limits

Same envelope and codes as boston-properties-search-site-content. No rate-limit headers are
returned and no limit is published — see rate-limits/boston-properties-rate-limits.yml. There is
no changelog, status page or deprecation policy for this surface; BXP could turn REST access off
without notice.
