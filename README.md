# My Blog

https://sunhwan.github.io/blog/

A personal blog by Sunhwan Jo, a computational chemist. Posts cover code and science.

## Built with

[Quarto](https://quarto.org). Posts are authored as `.qmd` files or Jupyter notebooks; Quarto renders to a static site that's published to GitHub Pages on every push to `master`.

(This site previously ran on [fastpages](https://github.com/fastai/fastpages); old `/YYYY/MM/DD/...html` URLs are preserved via per-post `aliases:` redirects.)

## Local preview

Install Quarto, then:

```bash
quarto preview      # live preview, auto-rebuild on save
quarto render       # one-shot full render into _site/
```

## Adding a post

Create `posts/<slug>/index.qmd` (or `index.ipynb`) with frontmatter:

```yaml
---
title: "Title"
description: "One-line description"
date: "YYYY-MM-DD"
categories: [foo, bar]
---
```

Drop any images or data the post references into the same folder. Notebook outputs are not re-executed at render time — save the `.ipynb` with its outputs already populated.
