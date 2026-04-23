# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal cybersecurity blog ("Project Tabula") built with Jekyll + Chirpy theme, deployed via GitHub Actions to GitHub Pages. Content is in Spanish and covers OpSec, Linux, and ethical hacking topics.

## Commands

```bash
# Local dev server (with live reload)
bash tools/run.sh

# Production build + html-proofer validation
bash tools/test.sh
```

VS Code tasks ("Run Jekyll Server", "Build Jekyll Site") wrap the same scripts.

## Architecture

**Theme:** `jekyll-theme-chirpy` (~7.4.1) via Gemfile; static assets are in `assets/lib/` as a git submodule.

**Content:**
- `_posts/` — blog articles named `YYYY-MM-DD-slug.md`, front matter uses `layout: post`, `categories`, `tags`, and optional `image`
- `_tabs/` — top-nav pages (`about`, `archives`, `categories`, `tags`); `order:` field controls menu position

**Data:**
- `_data/contact.yml` — social links
- `_data/share.yml` — per-post sharing buttons

**Plugin:** `_plugins/posts-lastmod-hook.rb` auto-injects `last_modified_at` from git commit history — no manual date update needed.

**Config:** `_config.yml` is the single source of truth for theme, language (`es`), timezone (`America/Santiago`), URL, author info, and feature flags (PWA, TOC, pagination).

## Deployment

Push to `main` triggers `.github/workflows/pages-deploy.yml`, which builds with `JEKYLL_ENV=production`, runs html-proofer, and deploys to GitHub Pages automatically.

## Conventions

- Indentation: 2 spaces; line endings: LF; charset: UTF-8 (enforced by `.editorconfig`)
- JS/CSS: single quotes; YAML: double quotes
- Post permalinks: `/posts/:title/`; tab permalinks: `/:title/`
- Markdown processor: Kramdown with Rouge syntax highlighting
