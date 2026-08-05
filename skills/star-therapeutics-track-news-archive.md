---
name: Track the Star Therapeutics news archive
description: Read Star Therapeutics' corporate news archive — press releases, news coverage and scientific presentations — over the public WordPress REST API, filtered by category and date window, with correct pagination.
api: openapi/star-therapeutics-content-openapi.yml
operations: [listCategories, listPosts, getPost]
method: generated
generated: '2026-08-05'
---

# Track the Star Therapeutics news archive

Star Therapeutics publishes its press releases, third-party news coverage and scientific
presentations as WordPress posts. The collection is small (28 items at harvest, 2022-02-16 through
2026-07-06) and readable without credentials. This is the only substantive dataset on the surface.

## Before you start

- Base URL: `https://star-therapeutics.com/wp-json`
- No authentication. Do not send an `Authorization` header — nothing accepts one.
- Send a browser-class `User-Agent`. The Cloudflare edge in front of the site answered non-browser
  agents with a 403 challenge during profiling. `/wp-json` responses were not challenged, but
  robots.txt asks for `Crawl-delay: 10` — pace yourself.
- Everything is public corporate communication. There is no patient data and no clinical data
  behind this API beyond what the press releases already say.

## Step 1 — resolve the categories (`listCategories`)

```
GET /wp/v2/categories?_fields=id,slug,name,count
```

Five terms are registered. At harvest: `press-release` (20 posts), `news-coverage` (7),
`presentations` (1), `our-perspective` (0), `uncategorized` (0). Take the `id` of the ones you
want — the posts endpoint filters by term id, not slug. Do not hardcode the ids; they are
deployment-specific.

The `post_tag` taxonomy is registered but empty, so `listTags` will return `[]`. Do not filter on
tags.

## Step 2 — list the posts (`listPosts`)

```
GET /wp/v2/posts?per_page=100&orderby=date&order=desc&_fields=id,slug,date,modified,link,title,excerpt,categories
```

- `per_page` is capped at **100**. Asking for more returns `400 rest_invalid_param` with the exact
  bound in `data.details.per_page.message` — clamp, do not retry blindly.
- Read `X-WP-Total` and `X-WP-TotalPages` from the response headers to know when to stop, and
  follow the `Link: …; rel="next"` header rather than incrementing `page` yourself.
- `_fields` matters here. The default representation carries the full rendered `content` plus a
  large `yoast_head` markup blob on every item; the field list above is roughly an order of
  magnitude smaller.

Narrow it:

- By category: `&categories=<id>` (or `&categories_exclude=<id>`).
- By date window: `&after=2025-01-01T00:00:00&before=2026-01-01T00:00:00` (ISO 8601).
- By what changed since your last poll: `&modified_after=<your last run>` — this is the right
  incremental-sync filter, not `after`, which is publication date.
- Full text: `&search=VGA039&search_columns=post_title,post_content`.

## Step 3 — fetch the body (`getPost`)

```
GET /wp/v2/posts/{id}
```

`content.rendered` is HTML, not markdown or plain text. `excerpt.rendered` is a short HTML summary.
`title.rendered` contains HTML entities and occasional inline markup (one 2026 title carries a
`<sup>` tag) — decode before comparing strings.

## Fields worth knowing

- `featured_image_src` and `featured_image_src_square` are direct image URLs inlined by the Avada
  theme. Use them instead of a second call to `/wp/v2/media/{id}` with `featured_media`.
- `author_info` carries the display data for the author. **`/wp/v2/users` returns 401**, so the
  numeric `author` id cannot be resolved — `author_info` is your only author signal.
- `yoast_head_json` carries a schema.org graph including a canonical URL, og: metadata and
  published/modified timestamps. Useful, but drop it with `_fields` unless you need it.

## Error handling

Errors are the WordPress envelope, **not** RFC 9457:

```json
{"code":"rest_post_invalid_id","message":"Invalid post ID.","data":{"status":404}}
```

Branch on `code`, not on `message` (which is localized). The ones you will actually hit:

| code | status | meaning |
|---|---|---|
| `rest_invalid_param` | 400 | A query parameter failed validation; read `data.details` |
| `rest_post_invalid_id` | 404 | No published post with that id |
| `rest_forbidden` | 401 | You touched an administrative route — back off, do not retry |

A `403` with an HTML body is Cloudflare, not the API. Slow down and use a browser `User-Agent`.

## Do not

- Do not attempt `/wp-json/jet-engine/v1/mcp` or `/wp-json/wp-abilities/v1/*`. Both are registered
  on this site and both return `401 rest_forbidden` anonymously. They are plugin defaults bound to
  an authenticated WordPress user, not an agent surface Star Therapeutics offers.
- Do not attempt any write. Every POST/PUT/PATCH/DELETE route requires an authenticated WordPress
  user, and this is not our site.
