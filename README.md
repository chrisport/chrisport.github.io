# chrisport.ch

Personal website and blog of Christoph Portmann, built with [Jekyll](https://jekyllrb.com/) and hosted on GitHub Pages at [chrisport.ch](https://chrisport.ch).

## Prerequisites

- Ruby 3.x and Bundler (`brew install ruby` on macOS)

Install dependencies once (gems go into `vendor/bundle`, which is gitignored):

```sh
bundle config set --local path vendor/bundle
bundle install
```

The `Gemfile` uses the `github-pages` gem, so local builds use the same Jekyll version and plugins as GitHub Pages.

## Run locally

```sh
bundle exec jekyll serve   # http://localhost:4000, rebuilds on change
```

Changes to `_config.yml` need a server restart.

## Build

```sh
bundle exec jekyll build    # outputs to _site/ (gitignored)
```

## Deploy

GitHub Pages builds and publishes the site automatically when you push to `master`:

```sh
git push origin master
```

The custom domain is set in `CNAME`. You can follow the build status under the repository's **Actions** tab on GitHub.

## Update dependencies

```sh
bundle update github-pages
```

Font Awesome is loaded from jsDelivr in `_includes/head.html`. To upgrade it, change the version in that URL.

## Adding content

**Blog post:** create `_posts/YYYY-MM-DD-slug.markdown`:

```yaml
---
layout: post
title:  "My post title"
date:   2026-01-31
categories: "golang"
status: "released"   # only "released" posts are listed; "draft" posts are still built and reachable by URL, with a [Draft] banner
---
```

**Project:** create `_code/YYYY-MM-DD-slug.markdown`. Projects appear on `/code/`:

```yaml
---
title: Project name
date:  2026-01-31
year:  2026
link:  https://github.com/chrisport/project
image: https://example.com/screenshot.png
---
One or two sentences describing the project.
```

## Structure

| Path | Purpose |
|---|---|
| `_config.yml` | Site settings |
| `index.html`, `1_posts.html`, `2_code.html`, `3_about.html` | Pages (number prefix = nav order) |
| `_posts/`, `_code/` | Blog posts and project entries |
| `_data/members.yaml` | Profile and social links used on About and in the footer |
| `_layouts/`, `_includes/` | Templates |
| `_sass/`, `css/` | Styles |
| `images/`, `fonts/` | Static assets |
