# Christoph Portmann - Personal Site

This is a Jekyll site (GitHub Pages compatible).

## Local Setup
1. Install Ruby (recommended via `rbenv`).
2. Install dependencies:
```bash
bundle install
```

## Run Locally
From the project root:
```bash
bundle exec jekyll serve --livereload
```

Open `http://localhost:4000` in your browser.

## Build
```bash
bundle exec jekyll build
```

The static site will be in `_site/`.

## Deployment
- GitHub Pages deploys from `master` using GitHub Actions (`.github/workflows/pages.yml`).
