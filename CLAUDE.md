# legionio.dev — LegionIO Marketing and Documentation Site

**Level 3 Documentation**
- **Parent**: `/Users/miverso2/rubymine/legion/CLAUDE.md`
- **GitHub**: https://github.com/LegionIO/legionio.dev
- **Live site**: https://legionio.dev

## Purpose

Marketing and documentation website for LegionIO. Targets developers evaluating whether to adopt LegionIO as an async job engine, agentic AI platform, or Ruby cognitive architecture. Published via GitHub Pages.

## Stack

- **Jekyll 4.3** — static site generator
- **Just the Docs 0.10** — documentation theme (dark color scheme)
- **jekyll-seo-tag** — meta/OG tags
- **GitHub Pages** — hosting (auto-deploys on push to main)

## Repository Layout

```
legionio.dev/
├── index.md                  # Home: hero + interactive visualization + navigation paths
├── architecture.md           # Technical architecture deep-dive
├── philosophy.md             # Design philosophy and motivation
├── extensions.md             # Extension ecosystem overview
├── settings.md               # Settings and configuration reference
├── compatibility.md          # Ruby and dependency compatibility matrix
├── enterprise.md             # Enterprise features, governance, compliance
├── getting-started/          # 3 quickstart guides (agentic, LLM, Ruby paths)
│   ├── quickstart-agent.md
│   ├── quickstart-llm.md
│   └── quickstart-ruby.md
├── assets/
│   ├── visualization.html    # Interactive SVG cognitive architecture hero animation
│   ├── favicon.svg           # SVG favicon (purple orbital motif)
│   ├── images/
│   │   ├── og-image.png      # OpenGraph share image (1200x630 PNG)
│   │   └── og-image.svg      # OG image source (SVG)
│   └── head_custom.html      # Unused (see _includes/)
├── _includes/
│   └── head_custom.html      # Injected into <head>: favicon link, custom meta
├── _config.yml               # Site config: title, theme, nav, search, footer
├── Gemfile                   # jekyll ~> 4.3, just-the-docs ~> 0.10, jekyll-seo-tag ~> 2.8
└── CNAME                     # legionio.dev
```

## Key Design Decisions

- **Hero**: The hero section uses an `<iframe>` embedding `assets/visualization.html` — an animated SVG showing cognitive modules and their connections. No JavaScript framework.
- **Three quickstart paths**: Agent (GAIA/cognitive), LLM (provider setup, pipeline), and Ruby developer (job engine basics). Each path promises an "aha moment" in 15 minutes.
- **No framework**: All custom JS is vanilla. The Just the Docs theme handles search, navigation, and layout.
- **OG image**: `/assets/images/og-image.png` — referenced in `_config.yml` as the default image for all pages.

## Local Development

```bash
bundle install
bundle exec jekyll serve
# http://localhost:4000
```

## Deployment

GitHub Pages auto-deploys on push to `main`. No CI workflow needed — Pages runs Jekyll natively.

## Content Governance

- Stats on the homepage (gem count, spec count, cognitive modules) should match actual shipped state.
- Current hero stats: `73 extension gems · 234 cognitive modules · 23,000+ specs` — update when these change significantly.
- Do not add external tracking scripts or analytics to this repo.

---

**Maintained By**: Matthew Iverson (@Esity)
