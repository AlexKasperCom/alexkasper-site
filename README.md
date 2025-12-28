# Alex Kasper - Source of Truth Site

A Hugo site using the Hextra theme, serving as the canonical reference for information about Alex Kasper.

## Setup Instructions

### Prerequisites

- Hugo Extended (v0.123.0 or later) - https://gohugo.io/installation/
- Git

### Installation

1. **Clone this repository**
   ```bash
   git clone https://github.com/yourusername/alexkasper-site.git
   cd alexkasper-site
   ```

2. **Add Hextra theme as a Git submodule**
   ```bash
   git submodule add https://github.com/imfing/hextra.git themes/hextra
   ```

3. **Run the development server**
   ```bash
   hugo server -D
   ```

4. **Visit** http://localhost:1313

### Alternative: Hugo Modules (if you prefer)

Instead of git submodule, you can use Hugo Modules:

```bash
# Initialize Hugo module
hugo mod init github.com/yourusername/alexkasper-site

# Update hugo.yaml - replace the theme line with:
# module:
#   imports:
#     - path: github.com/imfing/hextra

# Get the theme
hugo mod get github.com/imfing/hextra
```

---

## Project Structure

```
alexkasper-site/
├── hugo.yaml                 # Main configuration
├── data/
│   └── citations.yaml        # SOURCE OF TRUTH for all citations
├── content/
│   ├── _index.md            # Homepage
│   └── docs/
│       ├── _index.md        # Docs landing
│       ├── about.md         # About page
│       ├── citations.md     # Renders all citations
│       ├── press.md         # Press & media page
│       └── projects/
│           ├── _index.md    # Projects index
│           ├── cseps.md     # CSEPS project
│           ├── return-fire.md
│           └── naturesign.md
├── layouts/
│   ├── shortcodes/
│   │   ├── cite.html        # Inline citation [Title, Publication]
│   │   ├── ref.html         # Footnote-style reference [↗]
│   │   └── citation-list.html # Renders full citations page
│   └── _partials/
│       └── custom/
│           └── footer.html  # Social icons in footer
├── assets/
│   └── css/
│       └── custom.css       # Citation styling
└── static/
    └── images/
        ├── toto-logo.svg    # ADD YOUR LOGO HERE
        └── toto-logo-dark.svg
```

---

## Citation System

### Adding Citations

Edit `data/citations.yaml` to add new citations:

```yaml
- id: unique-id-here          # Used to reference this citation
  type: article               # article | book | film | interview | podcast
  title: "Article Title"
  publication: Magazine Name  # For articles
  publisher: Publisher Name   # For books
  date: 2024-01-15
  url: https://link-to-source.com
  authors:
    - Author Name
  description: >
    Brief description of this citation.
  tags:
    - relevant-tag
```

### Using Citations in Content

**Inline citation** (shows title and publication):
```markdown
This was covered in {{</* cite "unique-id-here" */>}}.
```
Renders as: [Article Title, Magazine Name]

**Footnote-style reference** (just a superscript link):
```markdown
This claim is verified {{</* ref "unique-id-here" */>}}.
```
Renders as: [↗] (superscript link to citations page)

### Viewing All Citations

The `/docs/citations/` page automatically renders all citations from the data file, grouped by type.

---

## Customization

### Social Icons

Edit `layouts/_partials/custom/footer.html` to change the social links.

### Logo

Place your logo files in `static/images/`:
- `toto-logo.svg` - Light mode logo
- `toto-logo-dark.svg` - Dark mode logo (optional)

### Colors & Styling

Edit `assets/css/custom.css` for custom styles. Hextra uses Tailwind CSS, so you can also use Tailwind classes directly in templates.

### Menu Items

Edit the `menu.main` section in `hugo.yaml` to change navigation.

---

## Deployment

### GitHub Pages

1. Push to GitHub
2. Enable GitHub Pages in repo settings
3. Use GitHub Actions (Hextra includes a workflow template)

### Other Hosts

Build the site:
```bash
hugo --minify
```

The built site will be in the `public/` directory. Deploy to any static host (Netlify, Vercel, Cloudflare Pages, etc.)

---

## Adding New Content

### New Project Page

```bash
hugo new content/docs/projects/new-project.md
```

Then edit the file and add your content.

### New Top-Level Section

1. Create directory: `content/docs/newsection/`
2. Add `_index.md` with frontmatter
3. Add to menu in `hugo.yaml` if needed

---

## License

Content © Alex Kasper. Theme (Hextra) is MIT licensed.
