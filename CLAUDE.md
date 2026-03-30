# legionio.dev: LegionIO Documentation Website

**Repository Level 3 Documentation**
- **Parent**: `/Users/miverso2/rubymine/legion/CLAUDE.md`

## Purpose

The public-facing documentation website for LegionIO, built with Jekyll and served via GitHub Pages. Contains architecture guides, getting-started content, extension development documentation, compatibility tables, settings reference, and the project philosophy.

**GitHub**: https://github.com/LegionIO/legionio.dev
**Live site**: https://legionio.dev (CNAME configured)

## Directory Structure

```
legionio.dev/
├── _config.yml              # Jekyll configuration (theme: just-the-docs, dark mode)
├── _includes/
│   └── head_custom.html     # OG meta tags, Twitter cards, hero CSS, favicon link
├── _site/                   # Generated output (git-ignored, do not edit)
├── assets/
│   ├── favicon.svg          # SVG favicon (purple "L" logo)
│   ├── images/
│   │   ├── og-image.svg     # OG image source (SVG)
│   │   └── og-image.png     # OG image (rasterized PNG for social cards)
│   └── visualization.html   # Interactive cognitive architecture visualization (SVG + JS particles)
├── getting-started/
│   ├── index.md             # Getting Started hub (audience routing table)
│   ├── quickstart-agent.md  # Cognitive Agent Quickstart (tick cycle, dreaming, 15 min)
│   ├── quickstart-ruby.md   # Extension Dev Quickstart (scaffold LEX gem, 10 min)
│   └── quickstart-llm.md   # LLM Routing Quickstart (multi-provider, Tier 0 cache, 10 min)
├── index.md                 # Home page (hero section with embedded visualization, install, quick links)
├── architecture.md          # Architecture deep-dive (system diagram, tick/dream cycles, cognitive domains)
├── philosophy.md            # Project philosophy and design principles
├── extensions.md            # Extension catalog (all 73 LEXs across 6 categories)
├── compatibility.md         # Ruby version and infrastructure compatibility matrix
├── enterprise.md            # Enterprise features (security, Vault, RBAC, deployment)
├── settings.md              # Comprehensive settings reference (all subsystems, every config key)
├── .github/
│   ├── workflows/
│   │   └── pages.yml        # GitHub Pages deployment (builds Jekyll, deploys to pages)
│   ├── CODEOWNERS           # Code ownership (@Esity for all files)
│   └── dependabot.yml       # Weekly updates for github-actions and bundler
├── CNAME                    # GitHub Pages custom domain: legionio.dev
├── Gemfile                  # Jekyll ~> 4.3, just-the-docs ~> 0.10, jekyll-seo-tag ~> 2.8
├── Gemfile.lock             # Dependency lock
├── README.md                # Basic project readme with dev instructions
└── .gitignore               # Ignores _site/, .sass-cache/, .jekyll-cache/, vendor/
```

## Key Pages

| Page | Path | nav_order | Audience |
|------|------|-----------|----------|
| Home | `index.md` | 1 | All visitors |
| Philosophy | `philosophy.md` | 2 | All visitors |
| Architecture | `architecture.md` | 3 | Developers |
| LLM Pipeline | `pipeline.md` | 4 | Developers |
| Extensions | `extensions.md` | 5 | Extension authors |
| Compatibility | `compatibility.md` | 6 | Integrators |
| Getting Started | `getting-started/` | 7 | New users |
| Enterprise | `enterprise.md` | 8 | Enterprise users |
| Settings Reference | `settings.md` | 9 | Operators, developers |

## Development

```bash
bundle install
bundle exec jekyll serve --livereload
# Site available at http://localhost:4000
```

## CI/CD

- GitHub Pages deployment via `.github/workflows/pages.yml`
- Triggers on push to `origin` branch or manual `workflow_dispatch`
- Builds with Ruby 3.4, Jekyll in production mode
- Dependabot watches github-actions and bundler weekly

## Design Notes

- Theme: just-the-docs (dark color scheme)
- Plugins: jekyll-seo-tag
- The hero section in `index.md` embeds `assets/visualization.html` — an animated SVG with rotating orbital rings and particle system (pure CSS animations + vanilla JS)
- `_includes/head_custom.html` provides OG/Twitter meta tags and all hero CSS styling
- Content on this site is user-facing marketing/docs, not AI-facing context. For AI-facing docs, see `/Users/miverso2/rubymine/legion/docs/`

---

**Maintained By**: Matthew Iverson (@Esity)
