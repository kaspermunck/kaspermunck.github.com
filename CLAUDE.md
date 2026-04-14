# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Jekyll 4.4 personal blog hosted on GitHub Pages, deployed via GitHub Actions. Ruby 3.3 is required (managed by rbenv).

## Prerequisites

rbenv must be initialised in the shell before any Ruby/Jekyll commands:

```bash
eval "$(rbenv init -)"
```

If rbenv is not installed: `brew install rbenv ruby-build && rbenv install 3.3.0`

## Common commands

```bash
# Install dependencies
bundle install

# Build the site (output to _site/)
bundle exec jekyll build

# Serve locally with auto-regeneration
bundle exec jekyll serve --watch

# Build including draft posts
bundle exec jekyll build --drafts
```

## Architecture

- **`_layouts/layout.html`** — base template; links CSS from `/assets/css/`
- **`_layouts/post.html`** — post template; extends `layout`
- **`_includes/`** — `analytics.html` (Google Analytics) and `footer.html` (social links)
- **`assets/css/`** — CSS source files (bootstrap.min, bootstrap-responsive.min, site, pygments). Edit here, not in the fingerprinted files elsewhere in `assets/`
- **`_drafts/`** — unpublished posts (not built unless `--drafts` flag is passed)
- **`2012/`, `2013/`, `2014/`** — pre-rendered HTML from the original blog; Jekyll copies them to `_site/` unchanged. No Markdown source exists for these.

## Deployment

Pushing to `master` triggers `.github/workflows/jekyll.yml`, which builds and deploys to GitHub Pages. Requires the GitHub Pages source to be set to "GitHub Actions" in the repo settings.

## Writing a new post

Create `_posts/YYYY-MM-DD-title.md` with front matter:

```yaml
---
layout: post
title: "Post Title"
categories: [ios, swift]
---
```

Post images go in `images/`.
