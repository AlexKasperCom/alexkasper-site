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
