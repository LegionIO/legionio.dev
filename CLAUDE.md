# docs-site: LegionIO Documentation Website

**Repository Level 3 Documentation**
- **Parent**: `/Users/miverso2/rubymine/legion/CLAUDE.md`

## Purpose

The public-facing documentation website for LegionIO, built with Jekyll and served via GitHub Pages. Contains architecture guides, getting-started content, extension development documentation, compatibility tables, and the project philosophy.

**GitHub**: https://github.com/LegionIO/docs-site
**Live site**: https://legionio.dev (CNAME configured)

## Directory Structure

```
docs-site/
├── _config.yml           # Jekyll configuration (theme: just-the-docs)
├── _includes/            # Shared partials (header, footer, nav)
├── _site/                # Generated output (do not edit, git-ignored)
├── assets/               # Static assets (CSS, JS, images, visualization.html)
├── getting-started/      # Getting-started guides (quickstart-agent.md, etc.)
├── index.md              # Home page (hero section, install, quick links)
├── architecture.md       # Architecture deep-dive
├── philosophy.md         # Project philosophy
├── extensions.md         # Extension ecosystem overview
├── compatibility.md      # Ruby version and gem compatibility matrix
├── enterprise.md         # Enterprise features and deployment guide
├── schema.md             # Settings JSON schema reference
├── CNAME                 # GitHub Pages custom domain
├── Gemfile               # Jekyll + just-the-docs
└── Gemfile.lock
```

## Key Pages

| Page | Path | Audience |
|------|------|----------|
| Home | `index.md` | All visitors |
| Architecture | `architecture.md` | Developers |
| Getting Started | `getting-started/` | New users |
| Extensions | `extensions.md` | Extension authors |
| Philosophy | `philosophy.md` | All visitors |
| Compatibility | `compatibility.md` | Integrators |
| Enterprise | `enterprise.md` | Enterprise users |

## Development

```bash
bundle install
bundle exec jekyll serve --livereload
# Site available at http://localhost:4000
```

## Design Notes

- Theme: just-the-docs (https://just-the-docs.com/)
- The hero section in `index.md` embeds `assets/visualization.html` — an interactive cognitive architecture visualization
- Content on this site is user-facing marketing/docs, not AI-facing context. For AI-facing docs, see `/Users/miverso2/rubymine/legion/docs/`

---

**Maintained By**: Matthew Iverson (@Esity)
