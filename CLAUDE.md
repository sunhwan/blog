# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal Quarto blog. Site is published at https://sunhwan.github.io/blog/ via GitHub Pages. Posts can be authored as `.qmd` (Quarto markdown) or `.ipynb` (Jupyter notebooks); Quarto renders them to a static site under `_site/`.

This repo was migrated from [fastpages](https://github.com/fastai/fastpages) (a Jekyll-based template that the fastai team deprecated in favor of Quarto). Old fastpages scaffolding is gone, but per-post `aliases:` preserve the original `/YYYY/MM/DD/<title>.html` URLs as redirect stubs so existing inbound links (including the inter-post link in the second AWS post) keep working.

## Local development

Quarto must be installed (`brew install --cask quarto` on macOS).

- `quarto preview` — live local preview at http://localhost:NNNN. Auto-rebuilds on file change.
- `quarto render` — full one-shot render into `_site/`.
- `quarto render <path/to/post>` — render a single post.

There is no separate test suite, linter, or formatter.

## Authoring conventions

Posts live in `posts/<slug>/`, each in its own folder with an `index.qmd` or `index.ipynb` and any local assets (images, data files) it references. The folder name is the URL slug (e.g., `posts/2021-04-17-run-md-on-aws-cluster/` → `https://sunhwan.github.io/blog/posts/2021-04-17-run-md-on-aws-cluster/`).

Required frontmatter for a post:

```yaml
---
title: "..."
description: "..."
date: "YYYY-MM-DD"
categories: [foo, bar]
aliases:
  - /YYYY/MM/DD/Old-Title.html   # only if migrating an existing fastpages URL
---
```

For notebook posts the YAML lives in a **raw cell** at the very top of the `.ipynb`. (Markdown-cell frontmatter does not work — Quarto only reads YAML from raw cells in notebooks.)

`posts/_metadata.yml` sets `execute: enabled: false`, so notebook code is **never re-executed at render time**; Quarto uses whatever output cells are saved in the `.ipynb`. If you add a new notebook post, run it once locally with its outputs saved before rendering. (This avoids needing RDKit, py3Dmol, etc. installed in CI.)

To collapse a code cell by default in a notebook, put `#| code-fold: true` as the first line of the code cell.

## Site config worth knowing

`_quarto.yml` is the source of truth:

- `site-url: https://sunhwan.github.io/blog` — the path prefix **must** be in `site-url` itself, not a separate `site-path` field (Quarto has no such field; setting it is silently ignored, and the sitemap/RSS/canonical URLs end up missing `/blog/`). Page-to-page navigation uses relative hrefs and works either way, so this bug is invisible locally.
- `resources: [assets/**]` — anything in that tree is copied verbatim into `_site/`. Cross-post static assets (the AWS post figures, downloadable example tarballs) live under `assets/` at the project root.
- `format.html.html-math-method: katex` — math is rendered with KaTeX.
- `execute: enabled: false` (in `posts/_metadata.yml`) — notebooks render from saved outputs only.

### Asset path gotcha

Quarto rewrites root-relative paths (`/foo/bar.png`) into file-relative ones (`../../foo/bar.png`) at render time. This means **do not include the `/blog` prefix in markdown image links** — write `![](/assets/figures/foo.png)`, not `![](/blog/assets/figures/foo.png)`. The `/blog` prefix is added by GitHub Pages at serve time, not by Quarto.

External `https://sunhwan.github.io/blog/...` URLs in prose (e.g., the `wget` example in the AWS post) are left as-is and continue to work in production.

## Deployment pipeline

`.github/workflows/publish.yml` is the single CI workflow. On push to `master` it runs `quarto-dev/quarto-actions/setup` then `quarto-dev/quarto-actions/publish` with `target: gh-pages`. The publish action renders the site and force-pushes the rendered output to the `gh-pages` branch using the auto-provided `GITHUB_TOKEN` — no separate SSH deploy key needed.

GitHub Pages is configured to serve from the `gh-pages` branch.
