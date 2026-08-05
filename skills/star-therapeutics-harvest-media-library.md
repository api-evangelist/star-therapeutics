---
name: Harvest the Star Therapeutics media library
description: Enumerate the 661-item media library over the public WordPress REST API, filter by media type and date, and pick the right generated size variant instead of downloading originals.
api: openapi/star-therapeutics-content-openapi.yml
operations: [listMedia, getMediaItem, listPosts]
method: generated
generated: '2026-08-05'
---

# Harvest the Star Therapeutics media library

`/wp/v2/media` exposes 661 attachments — corporate imagery, press-release assets and the figures
used across the site. Public, no credentials.

## Step 1 — enumerate (`listMedia`)

```
GET /wp/v2/media?per_page=100&orderby=date&order=desc&_fields=id,slug,date,modified,title,source_url,mime_type,media_type,filesize,post,alt_text
```

- `per_page` max is **100**; 661 items means 7 pages. Follow `Link: rel="next"` and stop when
  `X-WP-TotalPages` is reached.
- `_fields` is not optional at this size — the default representation inlines the whole
  `media_details.sizes` map for every item.

Filters that matter:

- `&media_type=image` — the enum is `image`, `video`, `text`, `application`, `audio`.
- `&mime_type=image/png` — narrower than `media_type`.
- `&parent=<post id>` — only attachments uploaded to a specific post or page.
- `&modified_after=<last run>` — the correct incremental-sync filter.
- `&search=<term>` — matches title, caption and description.

## Step 2 — pick the right variant (`getMediaItem`)

```
GET /wp/v2/media/{id}
```

`media_details.sizes` is a map of every generated variant (`thumbnail`, `medium`, `large`,
`full`, plus theme-specific sizes), each with its own `source_url`, `width`, `height`,
`mime_type` and `filesize`. **Take the smallest variant that satisfies your requirement.** The
top-level `source_url` is the original upload and is frequently many times larger than anything you
need.

`alt_text`, `caption.rendered` and `description.rendered` carry the descriptive text — that is
where the useful context lives for an image you cannot see. Many library items have empty
`alt_text`; do not assume it is populated.

## Step 3 — tie an asset back to its story

`post` is the id of the post or page the attachment was uploaded to, or `null` for library-only
uploads. Resolve it with `GET /wp/v2/posts/{id}` or `GET /wp/v2/pages/{id}`.

Going the other way, a post's `featured_media` is the attachment id — but the Avada theme already
inlines `featured_image_src` and `featured_image_src_square` on every post, so for featured images
you do not need this call at all.

## A gotcha worth knowing

Attachment `guid.rendered` values on this deployment point at WP Engine staging hostnames
(`startherastg.wpengine.com`, `startheradev2.wpenginepowered.com`), a leftover of the
staging-to-production workflow. **`guid` is an internal identifier, not a URL to fetch.** Always
download from `source_url` (or a `media_details.sizes[*].source_url`), which is on the production
host.

## Etiquette and errors

- robots.txt asks for `Crawl-delay: 10`. 661 items across 7 index pages is cheap; downloading 661
  originals is not. Fetch only what you need and pace the binary downloads.
- Send a browser-class `User-Agent` — the Cloudflare edge 403s aggressive non-browser agents on
  HTML paths.
- Errors are the WordPress envelope `{code, message, data:{status}}`. `rest_post_invalid_id` (404)
  for a bad attachment id; `rest_invalid_param` (400) for `per_page` over 100 or an unregistered
  `media_type` value.
