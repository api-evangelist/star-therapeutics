---
name: Search Star Therapeutics content and resolve the hits
description: Use the cross-content search endpoint to find Star Therapeutics material across posts, pages and media, then resolve each lightweight hit back to its full object.
api: openapi/star-therapeutics-content-openapi.yml
operations: [searchContent, getPost, getPage, getMediaItem, listPostTypes, listTaxonomies]
method: generated
generated: '2026-08-05'
---

# Search Star Therapeutics content and resolve the hits

`/wp/v2/search` searches across every published object in one call. It returns pointers, not
content — you resolve each one. Use this when you do not know whether a term lives in a press
release, a corporate page, or a media caption.

## Step 1 — search (`searchContent`)

```
GET /wp/v2/search?search=von+Willebrand&per_page=100
```

`search` is required. A query for `star` returned 68 matches at harvest. Each hit is:

```json
{"id": 3658, "title": "…", "url": "https://star-therapeutics.com/…", "type": "post", "subtype": "page"}
```

- `type` is the object class (`post`, `term`, `post-format`).
- `subtype` is the concrete post type — `post`, `page`, `attachment`, and so on. **This is the
  field that tells you which collection to resolve against.**
- Narrow with `&subtype=post` or `&subtype=page` when you only want one kind.
- Same pagination contract as every other collection: `page`, `per_page` (max 100), `X-WP-Total`,
  `X-WP-TotalPages`, `Link: rel="next"`.

## Step 2 — map subtype to a collection

Do not hardcode the mapping. Ask the API:

```
GET /wp/v2/types
```

Each entry carries `slug`, `rest_base` and `rest_namespace`. `rest_base` is the path segment to
use. Nineteen types are registered on this deployment; the ones that actually carry public content
are `post` (→ `posts`), `page` (→ `pages`) and `attachment` (→ `media`). The rest —
`avada_portfolio`, `avada_faq`, `elementor_library`, `elementor_snippet`, `elementskit_content`,
`elementskit_template`, `e-floating-buttons`, `jet-engine` and the WordPress internals — are either
empty or administrative.

`GET /wp/v2/taxonomies` gives you the same discovery for terms (eight registered: `category`,
`post_tag`, `nav_menu`, `wp_pattern_category`, `portfolio_category`, `portfolio_skills`,
`portfolio_tags`, `faq_category`).

## Step 3 — resolve each hit

```
GET /wp/v2/posts/{id}     # subtype: post   (getPost)
GET /wp/v2/pages/{id}     # subtype: page   (getPage)
GET /wp/v2/media/{id}     # subtype: attachment (getMediaItem)
```

All three ids share one identifier space, so a search hit id will never collide across these
collections. Add `?_fields=…` to trim; add `?_embed` to inline terms and featured media in the same
response.

Media objects give you `source_url` (the original file), `mime_type`, `filesize` and a
`media_details.sizes` map of every generated variant — take the smallest variant that meets your
need rather than the original.

## Cheaper alternative when you know the collection

Search is a good first move only when you do not know where the term lives. If you already know you
want news, `GET /wp/v2/posts?search=<term>&search_columns=post_title,post_content` is one request
instead of two and returns full objects directly.

## Errors

Same WordPress envelope as everywhere else. Branch on `code`:

- `rest_invalid_param` (400) — usually `per_page` over 100, or a `type`/`subtype` value that is not
  registered. Check `data.details`.
- `rest_post_invalid_id` (404) — the search index returned a pointer to something you cannot fetch
  in that collection; confirm you used the right `rest_base` for the `subtype`.
- `rest_forbidden` / `rest_user_cannot_view` (401) — you reached an administrative route. Notably
  `/wp/v2/users` is closed, so a search hit never resolves to an author record.
