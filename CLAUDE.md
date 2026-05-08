# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal Quarto blog. Site is published at https://sunhwanj.com via GitHub Pages (custom domain on the `sunhwan/blog` repo). Posts can be authored as `.qmd` (Quarto markdown) or `.ipynb` (Jupyter notebooks); Quarto renders them to a static site under `_site/`.

This repo was migrated from [fastpages](https://github.com/fastai/fastpages) (a Jekyll-based template that the fastai team deprecated in favor of Quarto). Old fastpages scaffolding is gone, but per-post `aliases:` preserve the original `/YYYY/MM/DD/<title>.html` URLs as redirect stubs so existing inbound links (including the inter-post link in the second AWS post) keep working.

## Local development

Quarto must be installed (`brew install --cask quarto` on macOS).

- `quarto preview` — live local preview at http://localhost:NNNN. Auto-rebuilds on file change.
- `quarto render` — full one-shot render into `_site/`.
- `quarto render <path/to/post>` — render a single post.

There is no separate test suite, linter, or formatter.

## Authoring conventions

Posts live in `posts/<slug>/`, each in its own folder with an `index.qmd` or `index.ipynb` and any local assets (images, data files) it references. The folder name is the URL slug (e.g., `posts/2021-04-17-run-md-on-aws-cluster/` → `https://sunhwanj.com/posts/2021-04-17-run-md-on-aws-cluster/`).

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

- `site-url: https://sunhwanj.com` — the site is served from the apex custom domain at root, no path prefix.
- `resources: [assets/**, CNAME]` — anything matched is copied verbatim into `_site/`. Cross-post static assets (the AWS post figures, downloadable example tarballs) live under `assets/` at the project root. **`CNAME` is the file that tells GitHub Pages this is the custom domain**; it must stay in `resources:` so Quarto copies it into `_site/CNAME` on every render. The publish workflow force-pushes `_site/` to `gh-pages`, so without this the custom-domain config would be wiped on the next deploy.
- `format.html.html-math-method: katex` — math is rendered with KaTeX.
- `execute: enabled: false` (in `posts/_metadata.yml`) — notebooks render from saved outputs only.

### Asset path gotcha

Quarto rewrites root-relative paths (`/foo/bar.png`) into file-relative ones (`../../foo/bar.png`) at render time, so `![](/assets/figures/foo.png)` resolves correctly from any post depth. That is the form to use in markdown image links.

Legacy external URLs of the form `https://sunhwan.github.io/blog/...` in prose (e.g., the `wget` example in the AWS post) are left as-is. GitHub Pages keeps serving the project-page URL after the custom domain switch and redirects it to `sunhwanj.com`, so those links continue to work in production.

## Deployment pipeline

`.github/workflows/publish.yml` is the single CI workflow. On push to `master` it runs `quarto-dev/quarto-actions/setup` then `quarto-dev/quarto-actions/publish` with `target: gh-pages`. The publish action renders the site and force-pushes the rendered output to the `gh-pages` branch using the auto-provided `GITHUB_TOKEN` — no separate SSH deploy key needed.

GitHub Pages is configured to serve the `gh-pages` branch with custom domain `sunhwanj.com` (Enforce HTTPS on). The custom-domain mapping persists across deploys via the `CNAME` file copied in by Quarto's `resources:` (see above).
