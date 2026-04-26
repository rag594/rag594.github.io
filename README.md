# raghavrastogi.in

Personal blog and portfolio site built with [Hugo](https://gohugo.io/) and the [hugo-blog-awesome](https://github.com/hugo-sid/hugo-blog-awesome) theme. Deployed to GitHub Pages via GitHub Actions on every push to `main`.

## Development

Requires Hugo Extended v0.160.1+.

```bash
# Serve locally with live reload
hugo server

# Build site to /public
hugo --minify
```

## Structure

```
content/
├── posts/          # Blog posts (page bundles)
├── about/          # Resume/career page
├── projects/       # Open source projects
└── archive/        # All posts by year

data/
├── career.toml     # Resume data (skills, experience, education)
└── projects.toml   # Projects data

layouts/
├── about/          # Custom career/resume template
├── archive/        # Custom archive template
└── projects/       # Custom projects template
```

## Deployment

Push to `main` → GitHub Actions builds with Hugo and pushes to the `gh-pages` branch → served at [raghavrastogi.in](https://raghavrastogi.in).
