# News & Media Archive Schema

This document defines the front matter schema used by the `/news` section of alexkasper.com.

Each news item is stored as a Hugo leaf bundle:

```text
content/news/example-entry/index.md
```

The folder name determines the public URL:

```text
/news/example-entry/
```

Dates are stored only in front matter and are not included in folder names or URLs.

The front matter is designed to:

* support articles, press clippings, YouTube videos, books, films, websites, blog posts, and other media;
* map cleanly to Wikipedia CS1 citation templates;
* separate archive metadata from citation metadata; and
* provide enough information for Hugo list pages and future export tools.

> **Principle:** Transcribe bibliographic facts only. Never invent missing information.

---

## Site Plumbing

### `title`

Title of the cited work or blog post.

### `date`

Original publication, release, or post date.

Use a complete ISO date:

```yaml
date: "1995-04-01"
```

The date is metadata only. It is not included in the folder name or public URL.

### `aliases`

Previous URLs that should redirect to the current page.

Aliases are stored on the current page, not at the old location.

Example:

```yaml
aliases:
  - /news/old-entry-name/
```

Multiple previous URLs may be retained:

```yaml
aliases:
  - /news/old-entry-name/
  - /news/earlier-entry-name/
```

Normally use an empty list when no aliases exist:

```yaml
aliases: []
```

Care must be taken not to create a new page whose URL conflicts with an existing alias.

---

## Classification

### `media_type`

Describes what the source itself is.

Allowed values include:

* article
* video
* audio
* podcast
* book
* book_chapter
* film
* television
* website
* webpage
* blog_post
* press_release
* document

### `entry_type`

Describes why the item belongs in the archive.

Examples:

* coverage
* profile
* interview
* appearance
* review
* mention
* authored
* created
* announcement
* reference

`entry_type: reference` has a functional effect: it excludes the item
from the homepage's "Latest News", the `/news/` list, and the "Latest
Posts" sidebar on other news pages. Use it for citation/lookup pages
that exist only to support a fact elsewhere on the site — Wikidata,
Wikipedia, IMDb, WorldCat, MobyGames, RocketReach, a company's own
About/homepage snapshot, a Secretary of State business filing — as
opposed to actual coverage, reviews, or announcements. Reference items
still appear in `{{< related-articles >}}` tables and are still
directly linkable and citable via `{{< fn >}}` / `{{< cite >}}`; they
just don't clutter "recent" surfaces alongside genuine news.

### `citation_type`

Controls which Wikipedia CS1 template is emitted.

Allowed values:

* news
* web
* book
* magazine
* journal
* av_media

---

## Site Content

### `summary`

Short original summary used on Hugo list pages and for SEO.

For multiline summaries:

```yaml
summary: >-
  This is the first line.
  This is the second line.
```

The lines are folded into a single paragraph.

### `projects`

List of related project slugs.

Example:

```yaml
projects:
  - return-fire
  - manufactured-urgency
```

### `tags`

Descriptive subject tags.

Example:

```yaml
tags:
  - review
  - behind-the-scenes
```

### `draft`

Set to `false` when ready to publish.

```yaml
draft: false
```

Do not automatically populate `lastmod` using `{{ .Date }}`. Either allow Hugo to derive it from Git or maintain it manually.

---

## Author / Creator

### `author`

Always use a YAML list.

People use:

```yaml
author:
  - "Kasper, Alex"
```

Multiple people use separate list entries:

```yaml
author:
  - "Hafner, Katie"
  - "Markoff, John"
```

Organizations use their literal name:

```yaml
author:
  - Associated Press
```

---

## Container & Publisher

### `work`

Container in which the work appeared.

Examples:

* The New York Times
* Computer Gaming World
* YouTube

### `publisher`

Publisher or platform when distinct from `work`.

### `channel`

Video or podcast channel when applicable.

### `via`

Database, archive, streaming service, or aggregator through which the source was consulted.

Examples:

* Newspapers.com
* Netflix

---

## Links & Archiving

### `source_url`

Canonical or original source URL.

The field is named `source_url` because Hugo reserves the `url` front matter field as a permalink override.

### `access_date`

Date the source was consulted.

```yaml
access_date: "2026-07-13"
```

### Primary archive

* `archive_url`
* `archive_date`
* `archive_service`

Example:

```yaml
archive_url: https://web.archive.org/example
archive_date: "2026-07-13"
archive_service: wayback
```

### Secondary archive

Optional fields for a second archived copy:

* `archive_url_2`
* `archive_date_2`
* `archive_service_2`

### `local_file`

Filename of the locally archived copy stored in the leaf bundle.

Examples include:

* PDF
* transcript
* image
* HTML snapshot

Example:

```yaml
local_file: 1995-04__en__Games-World_Return-Fire.pdf
```

### `prefer_local`

Optional. When `true`, the "Read/View" link on the single page always points at `local_file`, even when `source_url` (or an archive URL) is live.

```yaml
prefer_local: true
```

By default the link prefers the first live source in order: `source_url`, then `archive1_url`, then `archive2_url`, falling back to `local_file` only once all three are dead or unset. Set `prefer_local` when the local copy is the better reading experience than the live source — e.g. a periodical archive with its own corrected page order and OCR pass, like the TAP archive.

This is a separate flag from `source_dead` / `archive1_dead` / `archive2_dead`, which track actual link health and should stay accurate. Don't mark a live source dead just to force the local file to surface — set `prefer_local` instead.

---

## Print-specific Fields

### `pages`

Single page or page range.

Examples:

```yaml
pages: "51"
```

```yaml
pages: "51-53"
```

### `isbn`

ISBN for books.

Store the value as a string:

```yaml
isbn: "978-0-316-52858-0"
```

### Other optional fields

* `volume`
* `issue`
* `edition`
* `location`

---

## Audiovisual Fields

### `time`

Timestamp within the work where the cited material appears.

This is not the total runtime.

Example:

```yaml
time: "14:32"
```

### `duration`

Total runtime.

Examples:

```yaml
duration: "15:24"
```

```yaml
duration: "1:24:03"
```

### `cs1_type`

Optional override for CS1 `|type=`.

Examples:

* Video
* Motion picture

Normally omitted when `media_type` and `citation_type` already provide enough information.

---

## Language

### `language`

ISO 639-1 language code.

Examples:

* en
* fr
* de
* es
* ja
