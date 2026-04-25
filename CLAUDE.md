# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal blog and portfolio site at `https://raghavrastogi.xyz`, built with the [Zola](https://www.getzola.org/) static site generator and the Kita theme. Deployed to GitHub Pages via GitHub Actions on every push to `main`.

## Commands

### Site Development

```bash
# Serve locally with live reload (requires Zola installed)
zola serve

# Build site to /public
zola build

# Check for broken links
zola check
```

### Theme CSS (only needed when modifying `themes/kita/`)

```bash
cd themes/kita
pnpm dev    # Watch mode — recompiles Tailwind CSS on change
pnpm build  # One-time Tailwind compilation (app.css → main.css)
```

## Architecture

- **Zola** processes `config.toml` + `content/` + `themes/kita/` → static `public/`
- **GitHub Actions** (`.github/workflows/main.yml`) runs `shalzz/zola-deploy-action` on push to `main`, pushing the built site to the `gh-pages` branch
- The **Kita theme** is a git submodule at `themes/kita/`. Its templates live in `themes/kita/templates/`, its styles in `themes/kita/sass/` compiled via Tailwind CSS

## Content

Blog posts are Markdown files in `content/` with TOML front matter:

```toml
+++
title = "Post Title"
date = 2024-01-01
description = "Short description"
[taxonomies]
tags = ["tag1", "tag2"]
+++
```

Static pages live in `content/pages/` and use custom templates (`archive.html`, `projects.html`). Images and other assets for a post can be placed alongside its Markdown file or in `static/`.

## Configuration

All site-wide settings (base URL, theme, social links, nav menu, Atom feed, search index, taxonomies) are in `config.toml`. The Kita theme supports optional Giscus comments and KaTeX/Mermaid — these are configured via `[extra]` keys in `config.toml`.
