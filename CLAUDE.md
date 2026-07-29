# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Overview

- Personal blog at https://www.yduman.dev, built with Hugo (static site generator) using the `hugo-coder` theme.
- Content is Markdown with TOML front matter.
- Almost all work here is writing or editing posts — there is no application code.
- There are no tests, linters, or package manager.
- Local Hugo is v0.164.0 extended; the theme requires >= 0.124.0.

## Commands

```bash
hugo server -D          # local dev server with drafts, live reload (http://localhost:1313)
```

## Repository structure

- `content/` — all Markdown. `content/posts/*.md` are blog posts; `content/{about,uses,projects}.md` are the standalone menu pages.
- `hugo.toml` — site config: menu entries, social links, theme params. Adding a top-level page means adding both the file in `content/` and a `[[menu.main]]` block here.
- `assets/css/custom.css` — the only styling override, wired in via `params.customCSS`. Sets Inter for body text, Maple Mono (self-hosted from `static/fonts/`) for code, and dark/light background overrides on `body.colorscheme-dark|light`.
- `layouts/robots.txt` — a root-level override of the theme's template. `layouts/` is otherwise empty; the theme provides everything else.
- `static/` — copied verbatim to the site root. Post images live in `static/images/` and are referenced as `/images/foo.png`. `static/CNAME` is the custom-domain file for GitHub Pages — do not delete it.
- `themes/hugo-coder/` — **a git submodule**. Never edit files under it; changes are lost and CI checks out its own copy. To customize a theme template, copy it into `layouts/` at the same relative path (as done for `robots.txt`).
- `public/` — build output. Gitignored; never edit or commit by hand.

## Deployment

`.github/workflows/deploy.yml` runs on every push to `main`: checks out with `submodules: recursive`, builds with `hugo --minify`, and pushes `public/` to the separate `yduman/yduman.github.io` repo's `main` branch. Merging to `main` publishes — there is no staging step. The workflow needs the `ACTIONS_DEPLOY_TOKEN` and `DEPLOY_PRIVATE_KEY` secrets.
