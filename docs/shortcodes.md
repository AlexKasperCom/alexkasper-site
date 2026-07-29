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
