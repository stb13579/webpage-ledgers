# Reading Aikido pages out of Webflow

Verified against the live site. Read this before making Webflow calls, because two of the gotchas
below will produce confidently wrong findings if you don't know about them.

Site ID: `642adcaf364024552e71df01` (the only site on the account). The Webflow MCP tool names are
prefixed per-connection, so match on the suffix (`data_pages_tool`, `data_localization_tool`,
`data_assets_tool`, `get_asset_preview`) rather than a hardcoded prefix.

## Contents
- Before your first Webflow call
- Two sources of truth, and why you need both
- Resolving a URL to a page ID
- Reading live copy
- Reading Webflow nodes (hidden rot)
- Reading page metadata (SEO / OG)
- Reading images
- Token-cost warnings

## Before your first Webflow call

The MCP server requires a `webflow_guide_tool` call before any other Webflow tool, once per session.
It dumps roughly 60KB and will overflow to a file. That is expected. Do not read the file, you do not
need it, the recipes below are what it would have told you. Just make the call so the other tools
work, and move on.

Only needed if you actually call a Webflow tool. A batch scan working from an existing page inventory
never does, so skip it there.

One more fetching note, since it bites outside this site: fetching a competitor's documentation often
overflows to a file too. Grep the file for the capability you are checking rather than reading it.

## Two sources of truth, and why you need both

| Source | Shows | Use it for |
|---|---|---|
| the public URL, via `scripts/fetch_pages.py` | what a visitor sees, plus what is present but CSS-hidden, separated | judging live quality. Primary source for every copy-level finding. |
| `get_page_content` (Webflow) | every node in the page, including ones absent from the HTML entirely | the deepest rot: placeholder text, abandoned sections |

They disagree, and the disagreement is informative rather than a bug. Product pages on this site
carry placeholder text and whole abandoned sections in their nodes, left over from whichever page
they were duplicated from, and none of it reaches the HTML at all, let alone the screen. Go and look
rather than assuming any particular page is affected: reporting node content as live copy sends
someone to fix text no visitor can see, and inventing rot you did not actually find is worse.

There is a third state that is easier to get wrong than either of these, because it looks exactly
like live copy in raw HTML: **content present in the HTML and hidden by CSS.** Webflow uses
`display:none` on variant classes heavily on this site. `"No items found."` from an unbound CMS
collection and a testimonial captioned "GEA switched from Sonarqube to Aikido" both sit in the HTML
of many pages and neither is visible. `fetch_pages.py` exists to separate these, which is why the
skill routes page reads through it instead of extracting text by hand.

One nuance the script cannot settle for you: nav dropdown menus are also `display:none` until hover,
so they land in the hidden bucket while being genuinely visible to users. A stale product
enumeration in `_shared_chrome.hidden.txt` is a real finding, not rot.

## Resolving a URL to a page ID

Only needed when you have to touch Webflow (nodes, metadata, assets). If you only need live copy,
skip straight to the fetch script.

`list_pages` returns `publishedPath`, which is the full URL path including folder prefix. `slug` is
only the final segment, so matching on `slug` will collide across folders. Match on
`publishedPath`.

```
data_pages_tool > list_pages { site_id, limit: 100, offset: 0 | 100 | 200 | 300 }
```

There are ~323 pages, so each call is roughly 70k characters and **will** exceed the tool output
cap and be written to a file instead. That is fine and expected, though two batched actions in one
call can produce 150k+ and two overflow files, so make them one at a time. Parse the file with
python in Bash rather than reading it, and print only what you need:

```bash
python3 -c "
import json
d=json.loads(open('<saved-result-file>').read())
for p in d['result']['pages']:
    if '<path fragment>' in (p.get('publishedPath') or ''):
        print(p['id'], p['publishedPath'], p.get('draft'), p.get('isBranch'))
"
```

You rarely need all four offsets. If you are after one folder, fetch offsets until you have found
the prefix and the counts stop growing, rather than reconstructing the whole site every run. If an
inventory file from earlier the same day is available, spot-check a few IDs against a single
`list_pages` call instead of re-deriving all 323 pages.

Filter out `isBranch: true` (9 of them, staging branch duplicates) and note `draft: true` (32) so
you don't report findings on a page that isn't published. Pages with a `collectionId` are CMS
templates: one template drives many live URLs, so a finding there multiplies. `lastUpdated` is the
field to sort on when someone wants the stalest pages.

Known IDs, useful as a fast path but verify if a run depends on them:

| Path | Page ID |
|---|---|
| `/code/autofix` | `6734dac61e63e58cf6e19006` |
| `/comparison/aikido-vs-coderabbit` | `68a85438caba570183667950` |
| `/protect/device-protection` | `69d6080cf961e5e73b15b55a` |
| `/cloud/dspm` | `6a5621bff58a1589b6851031` |
| `/cloud/cloud-posture-management-cspm` | `65f2e16478f0b82120e5ccef` |

## Reading live copy

Use the bundled script. It curls the pages, resolves the visibility tiers, and writes clean text:

```bash
python3 /abs/path/to/skill/scripts/fetch_pages.py <url> [<url> ...] --outdir /tmp/aikido-pages
```

Per page you get `.visible.txt`, `.hidden.txt`, and `.images.tsv` (state, alt, CDN url). With four or
more pages you also get `_shared_chrome.visible.txt` and `_shared_chrome.hidden.txt`, which is the
nav and footer content lifted out so you review it once instead of paying for it on every page.

**Do not use `web_fetch` for these pages.** Two reasons. It deduplicates: if the URL was fetched
earlier in the session it replies "Already fetched Ns ago in this session" and returns **no content**,
which reads like a soft warning and is a total failure, and the documented `?v=1` cache-buster does
not reliably work around it. And its rendered markdown throws away the three things the audit needs
most: the `alt` attributes, the CDN image URLs, and the class names that reveal what is CSS-hidden.

If you need a one-off look at a single page and the script is unavailable, `curl` it directly and
strip it with python. Never reason about visibility from markdown.

```bash
curl -sSL "https://www.aikido.dev<publishedPath>" -o /tmp/page.html
```

## Reading Webflow nodes (hidden rot)

```
data_localization_tool > get_page_content { page_id, limit: 100, offset: 0 }
```

Omit `localeId` for the primary locale. Node types: `text`, `image`, `component-instance`,
`html-embed`. Visible text lives in `text.text` (plain) and `text.html` (with markup). Component
copy lives in `propertyOverrides[].text.text` or `.text.html`, each with a human-readable `label`
such as `"Title"`.

Three things to know:

1. **Component props left at their default are absent.** Only overridden props come back, so
   navbar, footer, logo strips, and default hero fallbacks are invisible here. Absence in this
   response is not evidence the page lacks something. Never raise an omission finding from node
   data alone, always confirm against the page's `.visible.txt`.
2. **`pagination.total` under-counts.** Page until `nodes` comes back shorter than `limit` rather
   than trusting the count.
3. **FAQ answers are CMS-bound** and return "No items found." here. They do render live, so
   read them from `.visible.txt`. Note that a *separate* "No items found." string is CSS-hidden in
   the HTML of many pages from an unbound testimonial collection. Same words, different problem.

The AutoFix page is 62 nodes and about 30k characters in one call. Survivable for a deep dive, too
expensive to do for every page in a batch, which is why batch mode skips this read by default.

## Reading page metadata (SEO / OG)

```
data_pages_tool > get_page_metadata { page_id }
```

Returns `seo{title,description}` and `openGraph{title,titleCopied,description,descriptionCopied,
imageUrl,imageAssetId}`. Cheap. `titleCopied: true` means OG inherits from SEO, which is normal and
not a finding. A missing or truncated `seo.description`, or an OG image that is null on a page
built to be shared, is worth a low-severity note.

## Reading images

**Start from the live HTML, not from Webflow.** The rendered page already contains every CDN image
URL and every `alt` attribute, which is everything the asset checks need. Going through
`get_asset` for a `hostedUrl` you already have is a wasted round trip, and the node-level
`__wf_reserved_inherit` plus `altText: null` inference is a weaker answer than reading `alt=""`
directly off the page. Use the Webflow asset calls only when the live HTML does not cover it, for
instance checking an OG image that is not rendered in the body.

Asset IDs, when you do need them, come from `image.assetId` on image nodes, or
`openGraph.imageAssetId`.

```
data_assets_tool > get_asset { asset_id }
```

Returns `originalFileName`, `contentType`, `size`, `hostedUrl`, `altText`, and
`variants[{hostedUrl,format,width,height,quality}]`.

Two traps: `variants[].height` is always `null` and the base asset carries no dimensions, so you
cannot judge resolution from this response. And `alt: "__wf_reserved_inherit"` on the node means
"inherit from the asset," so the real alt text is the asset's `altText`, which is often `null`,
meaning the image has no alt text at all.

For actual resolution and actual visual review, use the bundled script, which downloads the
original from `hostedUrl`, reports true dimensions, and converts to PNG so you can look at it:

```bash
python3 /absolute/path/to/skill/scripts/inspect_asset.py <url> [<url> ...] --outdir /tmp/aikido-assets
```

Use an absolute path. The working directory does not persist between bash calls, so a relative path
costs you a round trip finding it again.

`get_asset_preview { asset_id }` also works but returns the smallest variant (500px avif), which is
too small to judge whether a screenshot is pixelated and is not directly viewable. Prefer the
script when resolution is the question.

## Token-cost warnings

- `list_pages` at limit 100: ~70k chars, always overflows to a file. Parse with python.
- `get_page_content` on a large page: ~30k chars. Deep dive only.
- `data_element_tool > query_elements` with `component_filter`: enormous output, one hero instance
  dumped roughly 40 props with full slot trees. Avoid. With `element_filter` it is cheap and gives
  heading structure, but note that headings rendered from component props do not appear as
  `Heading` elements, so a heading audit built only on this will miss the H1.
- `data_comments_tool > list_comment_threads`: currently returns zero threads for this site, so
  Webflow Designer comments are not a feedback source today. Worth one cheap check in a deep dive
  in case that changes.
