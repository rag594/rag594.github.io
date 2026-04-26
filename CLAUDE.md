# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal blog and portfolio site at `https://raghavrastogi.in`, built with [Hugo](https://gohugo.io/) and the [hugo-blog-awesome](https://github.com/hugo-sid/hugo-blog-awesome) theme. Deployed to GitHub Pages via GitHub Actions on every push to `main`.

## Commands

### Site Development

```bash
# Serve locally with live reload (requires Hugo Extended v0.160.1+)
hugo server

# Build site to /public
hugo --minify
```

## Architecture

- **Hugo** processes `hugo.toml` + `content/` + `themes/hugo-blog-awesome/` → static `public/`
- **GitHub Actions** (`.github/workflows/`) builds with Hugo on push to `main` and pushes the built site to the `gh-pages` branch
- The **hugo-blog-awesome** theme is a git submodule at `themes/hugo-blog-awesome/`

## Content

Blog posts are Markdown files in `content/posts/` with YAML or TOML front matter. Static pages live in `content/about/`, `content/projects/`, and `content/archive/`.

Resume/career data is in `data/career.toml` and projects data is in `data/projects.toml`, rendered via custom templates in `layouts/`.

## Configuration

All site-wide settings (base URL, theme, social links, nav menu, RSS) are in `hugo.toml`.
