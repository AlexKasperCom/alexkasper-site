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

## Citations & footnotes (`cite`, `fn`, `references`)

Wikipedia-style citations: a bracketed, superscript number in prose — `[1]` — linked to a numbered, formatted reference list at the bottom of the page, with backlinks from each list entry to every place it's cited.

Templates:

* `layouts/partials/cite-format.html` — the actual citation formatter (shared by both paths below)
* `layouts/shortcodes/cite.html` — manual path
* `layouts/shortcodes/fn.html` + `layouts/shortcodes/references.html` — auto-collected path
* `layouts/partials/footnote-backrefs.html` — JS included site-wide, only relevant to the manual path (see below)

There are **two ways to cite something**, and they render identically — pick whichever fits how you're writing.

### Path 1: `fn` + `references` (auto-collected — use this one)

Drop a citation marker in prose:

```text
CSEPS was delivered as a two-day course.{{< fn "n000069" >}}
```

and once, near the bottom of the page, after every citation it should cover:

```text
{{< references >}}
```

`{{< references >}}` takes no arguments — it reads back whatever `{{< fn >}}` calls appeared earlier in the *same page* and renders the full list from that, in first-citation order. There's nothing else to maintain: add, remove, or reorder `{{< fn >}}` calls in the prose and the list at the bottom follows automatically.

This works by writing to the page's build-time `.Store` as each `{{< fn "n000069" >}}` renders (first-seen order, plus a per-id usage count for backlinks), which `{{< references >}}` then reads back. Because of that, **`{{< references >}}` must come after every `{{< fn >}}` call it's meant to include** — Hugo renders shortcodes in document order, so a `{{< references >}}` placed too early would simply miss later citations. In practice this just means: keep it at the bottom, like a normal references section.

### Path 2: `cite` + native Markdown footnotes (manual)

The older path, still supported, useful when you want a specific footnote label with its own locator (see below) rather than one shared entry per source:

```text
CSEPS was delivered as a two-day course.[^n000069]

[^n000069]: {{< cite "n000069" >}}
```

This pairs `cite` with **Hugo's built-in Goldmark footnote extension**, which is on by default (no config needed) — standard Markdown `[^label]` / `[^label]: ...` syntax. The definitions can live anywhere in the file; by convention, group them at the bottom. The trade-off versus Path 1: you're maintaining two things in sync by hand — the `[^label]` markers in prose and the matching `[^label]: {{< cite ... >}}` definitions.

Reuse the same label to cite a source more than once:

```text
Mitnick and Kasper developed CSEPS in 2003.[^n000069] ... It was later delivered nationwide.[^n000069]
```

### Both paths: citing the same source twice

Whichever path you use, citing the same source more than once on a page collapses to **one** numbered entry in the list, with one backlink per place it's cited — Wikipedia's "^ a b c" convention. A source cited once gets a single linked "^"; cited more than once, "^" becomes a plain (non-linked) marker followed by lettered superscript links (a, b, c, ...), one per citation, each jumping back to that specific spot in the prose.

For Path 1 (`fn`/`references`) this is built server-side in `references.html`, straightforwardly, since the shortcode controls the HTML directly.

For Path 2 (native `[^label]` footnotes), Goldmark always renders backlinks as plain trailing arrows (`↩`), one per usage, at the *end* of the entry — Hugo exposes no render hook for footnotes, so there's no template-level way to change that shape. `footnote-backrefs.html` (included site-wide from `baseof.html`, a no-op on any page without a `.footnotes` block) is a small vanilla-JS pass that runs after page load and rewrites that trailing-arrows markup into the same "^ a b c" group at the *front* of the entry, to match Path 1's output. If JS is disabled, the original trailing-arrow version still works — `.footnote-backref` in `ak.css` is kept as that fallback.

### Citation format (shared by both paths)

`cite-format.html` renders Wikipedia CS1-style punctuation:

```text
With an author:     Author (Date). "Title". Publisher. Locator.
Without an author:  "Title". Publisher. Date. Locator.
```

* Title is linked to the cited page. Quoted for most `source_type`s (article, webpage, newsletter, press-release, ...); **italicized instead of quoted** for `source_type: book` or `source_type: document` — a standalone work rather than something published inside a container publication (matches how Wikipedia itself formats book and internal-document citations, e.g. the CSEPS Training Workbook entries in `content/projects/p000004/index.md`).
* If the cited page's `author` is the same organization as its `publisher` (an internal document whose only "author" is the company that produced it), the author is dropped rather than printed twice.
* Set `date_precision: year` or `date_precision: month` in the cited page's front matter when only the year, or only the month, of publication is known (its `date` field still needs a full `YYYY-MM-DD` value, e.g. `1995-04-01`, so it still sorts correctly) to display "1995" or "April 1995" instead of a fabricated "April 1, 1995". `month` is the common case for print magazine issues — a cover-dated month with no specific day.
* An optional second argument to `cite` (`{{< cite "n000080" "pp. 97–99" >}}`) appends a locator — page numbers, a timestamp — specific to that one citation, without touching the cited page's own `pages` field. Only available on Path 2, since Path 1 keys purely by id (citing the same id twice always collapses into one entry, with no way to attach a different locator to each usage).

### Styling

In `ak.css`:

* `.footnote-ref::before`/`::after` add the `[` `]` brackets around the inline superscript number; `sup[id^="fnref"] { line-height: 0; }` stops the raised glyph from expanding the line box it sits on (otherwise lines with a citation get visibly different leading from lines without one).
* `.footnotes` sizes and tightens the reference list itself (smaller and denser than body copy — it's a reference list, not prose) and hides the `<hr>` Goldmark inserts above it (redundant once there's already a `#### References`-style heading above the list).
* `.fullPost h1`–`h6` puts a thin rule under every heading inside page content (`.fullPost` is the shared body-copy wrapper — see `layouts/*/single.html` templates), Wikipedia-style — scoped so it never reaches nav/header/footer/sidebar headings that have their own unrelated styling.

### `cite`/`fn` vs. `ref-id`

Use `cite` or `fn` for anything footnoted as the source for a specific claim. Keep using plain `{{< ref-id >}}` for ordinary in-body links to other pages that aren't functioning as a citation (e.g. "see `{{< ref-id "n000003" >}}` for more on this").

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

Renders a grid of labeled photos from the current page's `photos` front matter. Each real file an entry has — `full` and/or `square` — gets its own tile; clicking a tile downloads that exact file.

Template: `layouts/partials/photo-gallery.html` (the actual rendering logic), with `layouts/shortcodes/photo-gallery.html` as a thin wrapper so it can also be dropped into a markdown body.

### Why it exists

Pages like `/press` need a simple, labeled photo grid (headshots, event photos) without hand-writing image markup. Originally shortcode-only; moved to a partial so a page with its own dedicated layout (like `layouts/press/single.html`) can call it directly via `{{ partial "photo-gallery.html" . }}` instead of needing a markdown body just to host a shortcode. Front-matter-driven, like the `about` page's carousel, but for a plain grid rather than a slider.

### Usage

From a layout template:

```go-html-template
{{ partial "photo-gallery.html" . }}
```

Or inline in any markdown body:

```text
{{< photo-gallery >}}
```

Both read from the page's own front matter — no arguments:

```yaml
photos:
  - label: "Studio headshot"
    full: "ak-portrait-1.jpg"
    square: "ak-portrait-1-square.jpg"
    alt: "Alex Kasper, studio headshot"

  - label: "Speaking photo"
    full: "ak-speaking-1.jpg"
    square: "ak-speaking-1-square.jpg"
    alt: "Alex Kasper speaking on stage"
```

### Required front matter fields

Each entry under `photos`:

* `label` — caption shown under each tile (e.g. "Studio headshot").
* `full` — the page-resource filename for the full-resolution original.
* `square` — optional page-resource filename for a pre-cropped square version. Omit it if no crop exists for that photo.
* `alt` — alt text for the tile's image.

At least one of `full`/`square` must be a page resource (a file sitting alongside `index.md` in the same page bundle) — the partial resolves each with `Resources.GetMatch` and silently skips any version whose file isn't found. There's no separate "web" file to maintain: the on-page preview shown in the grid is generated on the fly by Hugo's image processing (`Resource.Fill`) from whichever real file it's previewing.

### Output

Renders a CSS grid with one tile per available version — clicking a tile opens that exact file:

```html
<div class="photo-gallery">
  <figure class="photo-gallery__item">
    <a href="/press/ak-portrait-1.jpg" target="_blank" rel="noopener">
      <img src="/press/ak-portrait-1_hu1234.jpg" alt="Alex Kasper, studio headshot" width="480" height="480" loading="lazy">
    </a>
    <figcaption>
      <span class="photo-gallery__label">Studio headshot</span>
      <span class="photo-gallery__version">Full size</span>
    </figcaption>
  </figure>
  <figure class="photo-gallery__item">
    <a href="/press/ak-portrait-1-square.jpg" target="_blank" rel="noopener">
      <img src="/press/ak-portrait-1-square_hu5678.jpg" alt="Alex Kasper, studio headshot" width="480" height="480" loading="lazy">
    </a>
    <figcaption>
      <span class="photo-gallery__label">Studio headshot</span>
      <span class="photo-gallery__version">Square crop</span>
    </figcaption>
  </figure>
  ...
</div>
```

A small `<style>` block scoped to `.photo-gallery` is emitted alongside the grid by the partial itself, so both the shortcode and direct `partial` calls stay self-contained.

### Adding new photos

Drop the full-resolution image (and, if you have one, a pre-cropped square version) into the page's bundle directory, and add an entry under `photos` in front matter — no resizing, no separate web copy, no changes needed to the partial or shortcode.
