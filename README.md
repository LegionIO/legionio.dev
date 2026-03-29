# legionio.dev

The marketing and documentation website for LegionIO, published at https://legionio.dev.

Built with Jekyll + [Just the Docs](https://just-the-docs.com/) theme. Deployed via GitHub Pages.

## Development

```bash
bundle install
bundle exec jekyll serve
# open http://localhost:4000
```

## Structure

```
legionio.dev/
├── index.md              # Home page with hero section and cognitive architecture visualization
├── architecture.md       # Technical architecture deep-dive
├── philosophy.md         # Why LegionIO was built
├── extensions.md         # Extension ecosystem overview
├── settings.md           # Settings and configuration reference
├── compatibility.md      # Ruby and dependency compatibility matrix
├── enterprise.md         # Enterprise features and governance
├── getting-started/      # Getting started guides (3 paths)
│   ├── quickstart-agent.md  # Agentic AI path
│   ├── quickstart-llm.md    # LLM integration path
│   └── quickstart-ruby.md   # Ruby developer path
├── assets/
│   ├── visualization.html   # Interactive cognitive architecture visualization (hero)
│   ├── favicon.svg          # SVG favicon
│   ├── images/
│   │   ├── og-image.png     # OpenGraph image (1200x630)
│   │   └── og-image.svg     # OG image source
│   └── head_custom.html     # Jekyll head customization (injected via _includes)
├── _includes/
│   └── head_custom.html     # Custom head HTML (favicon link, meta tags)
├── _config.yml              # Jekyll + Just the Docs configuration
├── Gemfile                  # jekyll, just-the-docs, jekyll-seo-tag
└── CNAME                    # legionio.dev (GitHub Pages custom domain)
```

## Deployment

Deployed automatically by GitHub Pages on push to `main`. No build step required — Jekyll runs server-side.

The `CNAME` file sets the custom domain to `legionio.dev`.

## License

Apache-2.0
