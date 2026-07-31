# Shortcodes

Custom Hugo shortcodes used across alexkasper.com.

---

## `ref-id`

Links to any content item (`news`, `projects`, `events`) by its front-matter `id`, without needing to know its slug or URL.

Template: `layouts/shortcodes/ref-id.html`

### Why it exists

Every leaf bundle in `/news`, `/projects`, and `/events` has a stable `id` in front matter that matches its folder name:

```yaml
id: n000178
```

```text
content/news/n000178/index.md
```

Slugs and titles can change; `id` never does. `ref-id` resolves a link at build time by searching `site.RegularPages` for a matching `Params.id`, so links stay correct even if the target page is renamed or moved between sections.

### Usage

Inline in any markdown body:

```text
See {{< ref-id "n000003" >}} for more on this.
```

Renders as a link to the matching page, using its title as the link text:

```html
See <a href="/news/n000003/cyberpunk-outlaws-and-hackers-on-the-computer-frontier/">Cyberpunk: Outlaws and Hackers on the Computer Frontier</a> for more on this.
```

### Custom link text

Pass a second argument to override the default (title) link text:

```text
Check out {{< ref-id "n000003" "this book" >}} for more.
```

```html
Check out <a href="/news/n000003/cyberpunk-outlaws-and-hackers-on-the-computer-frontier/">this book</a> for more.
```

### Error handling

If no page has a matching `id`, the build fails with an explicit error naming the bad ID and the file it was used in, rather than silently producing a dead link:

```text
ref-id shortcode: no page found with id "n999999" (used in "content/news/n000121/index.md")
```

### How the lookup works

```go-html-template
{{- $matches := where site.RegularPages "Params.id" $id -}}
```

This scans every regular page on the site (all sections) for one whose `Params.id` equals the given string, and takes the first match. Same `where site.RegularPages` pattern already used elsewhere in this codebase (e.g. `layouts/_default/home.html`, `layouts/events/archive.html`) for section listings.

### Adding new ID'd content types

Any new section works automatically as long as its front matter includes an `id` field — no changes needed to the shortcode itself.

---

## `back-issues`

Renders a table listing every page tagged with a given project slug, sorted by issue number, each row linking to its page.

Template: `layouts/shortcodes/back-issues.html`

### Why it exists

Numbered periodical archives (e.g. TAP) carry an `issue` field and a descriptive `title` in front matter (e.g. `"TAP No. 28 - Reading Computer Bills, Loop Suffixes"`). This shortcode turns that data into a browsable index on the parent project page, in the spirit of the "Back Issues - Listed by Feature Articles" catalog the source publication itself used to print.

### Usage

Inline in any markdown body:

```text
{{< back-issues "tap-archive" >}}
```

The argument is a project slug — it matches pages whose front-matter `projects` list includes that slug:

```yaml
projects:
  - tap-archive
```

The slug must match the parent project page's own `slug` field exactly (e.g. `content/projects/p000008/index.md` has `slug: tap-archive`) — `visit-project.html` (see below) relies on that same match to link an entry back to its project page, so a mismatch silently breaks both.

### Required / used front matter fields

* `projects` — must include the slug passed to the shortcode, and must match a project page's `slug` for `visit-project.html` to find it. Used to select which pages appear.
* `issue` — the issue label shown in the "No." column (can be numeric, like TAP's `1`-`91`, or a string, like Classified TAP's `"C1"`-`"C7"` — sorting doesn't depend on it, so mixed types across a sub-series are fine).
* `title` — used directly as the link text, so it should already read the way you want it displayed (e.g. `"TAP No. 28 - Reading Computer Bills, Loop Suffixes"`).
* `date` — used both to sort the table and to render the "Date" column (formatted `Jan 2006`); omitted from the row if not set.

An optional second argument to the shortcode narrows the table to one `series` value, for projects that host more than one independently-numbered sub-series under the same project slug (see `series-nav.html` above, which handles the prev/next side of the same problem).

### Output

Renders an HTML table with columns No. / Date / Title:

```html
<table class="back-issues-table">
  <tbody>
    <tr>
      <td class="issue-no">1</td>
      <td class="issue-date">Jun 1971</td>
      <td class="issue-feature"><a href="/news/n000121/tap-001/">TAP No. 1 - Extensions, Conference Switches</a></td>
    </tr>
    ...
  </tbody>
</table>
```

A small `<style>` block scoped to `.back-issues-table` is emitted alongside the table (the site's global stylesheet has no table styling of its own).

### How the lookup works

```go-html-template
{{- $pages := where site.RegularPages "Params.projects" "intersect" (slice $slug) -}}
{{- $pages = sort $pages "Date" "asc" -}}
```

Same `where site.RegularPages` pattern as `ref-id` above, filtered by the `projects` list and sorted by publication date — not by `issue` — so a sub-series with its own numbering (like the Classified TAP ad sheets) can interleave correctly among the numbered issues instead of colliding with them.

### Adding new periodical archives

Any project can use this shortcode as long as its issues carry `projects` (matching the slug passed to the shortcode), `issue`, and a `title` that already reads the way you want it linked — no changes needed to the shortcode itself.

---

## `photo-gallery`

Renders a grid of thumbnails from the current page's `photos` front matter, each linking to the full-resolution image.

Template: `layouts/shortcodes/photo-gallery.html`

### Why it exists

Pages like `/press` need a simple photo grid (headshots, event photos) without hand-writing image markup or wiring up a new layout per page. Front-matter-driven, like the `about` page's carousel, but for a plain grid rather than a slider.

### Usage

Inline in any markdown body:

```text
{{< photo-gallery >}}
```

Reads its data from the page's own front matter — no arguments:

```yaml
photos:
  - image: "ak-portrait-1.jpg"
    thumb: "ak-portrait-1-square.jpg"
    alt: "Alex Kasper, studio portrait"

  - image: "ak-speaking-1.jpg"
    thumb: "ak-speaking-1.jpg"
    alt: "Alex Kasper speaking on stage"
```

### Required front matter fields

Each entry under `photos`:

* `image` — the full-resolution page-resource filename; the thumbnail links here.
* `thumb` — the page-resource filename actually displayed in the grid. Use a separate cropped file for a tighter square thumbnail, or repeat the same filename as `image` to use the full image as its own thumbnail.
* `alt` — alt text for the thumbnail.

Both `image` and `thumb` must be page resources (files sitting alongside `index.md` in the same page bundle) — the shortcode resolves them with `Resources.GetMatch` and silently skips any entry where either file isn't found.

### Output

Renders a CSS grid of square-cropped thumbnails, each wrapped in a link to the full-resolution original (opens in a new tab). When `thumb` differs from `image`, both versions are offered as separate download links below the thumbnail — "Square crop" and "Original" — rather than only exposing the original through the image link. When `thumb` and `image` are the same file, a single "Full size" link is shown instead:

```html
<div class="photo-gallery">
  <figure class="photo-gallery__item">
    <a href="/press/ak-portrait-1.jpg" target="_blank" rel="noopener">
      <img src="/press/ak-portrait-1-square.jpg" alt="Alex Kasper, studio portrait" width="2000" height="2000" loading="lazy">
    </a>
    <figcaption class="photo-gallery__links">
      <a href="/press/ak-portrait-1-square.jpg" target="_blank" rel="noopener">Square crop</a>
      <a href="/press/ak-portrait-1.jpg" target="_blank" rel="noopener">Original</a>
    </figcaption>
  </figure>
  ...
</div>
```

A small `<style>` block scoped to `.photo-gallery` is emitted alongside the grid.

### Adding new photos

Drop the image file(s) into the page's bundle directory, add an entry under `photos` in front matter, and (optionally) crop a square thumbnail alongside the original — no changes needed to the shortcode itself.
