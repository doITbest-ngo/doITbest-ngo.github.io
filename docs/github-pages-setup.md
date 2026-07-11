# GitHub Pages Setup

## Prerequisites

- GitHub account with access to the `doITbest-ngo` organization
- `gh` CLI installed and authenticated (`gh auth status`)
- Git installed

## 1. Create/Rename Repository

The repository **must** be named `doITbest-ngo.github.io` for the root site URL.

```bash
# Via GitHub CLI (requires admin access on org)
gh repo rename doITbest-ngo.github.io -R doITbest-ngo/doITbest-ngo -y

# Or rename manually: Settings → Repository name
```

**URL**: `https://doITbest-ngo.github.io/`

> Project sites (unlimited) use `<org>.github.io/<repo-name>` URLs.

## 2. Clone & Setup Remote

```bash
gh repo clone doITbest-ngo/doITbest-ngo.github.io
cd doITbest-ngo.github.io
git remote set-url origin git@github.com:doITbest-ngo/doITbest-ngo.github.io.git
```

## 3. Jekyll File Structure

```
_config.yml          # Site configuration
Gemfile              # Ruby dependencies
index.md             # Landing page
_layouts/
  default.html       # Base HTML template
_posts/              # Blog posts (empty, ready for content)
.github/
  workflows/
    pages.yml        # GitHub Actions deploy workflow
```

### _config.yml

```yaml
title: doITbest-ngo
description: doITbest NGO website
url: "https://doITbest-ngo.github.io"
baseurl: ""

markdown: kramdown
plugins:
  - jekyll-feed
  - jekyll-seo-tag

exclude:
  - Gemfile
  - Gemfile.lock
  - node_modules
  - vendor
```

### Gemfile

```ruby
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins
```

### .github/workflows/pages.yml

```yaml
name: Build and deploy Jekyll to Pages

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: pages
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: "3.3"
          bundler-cache: true

      - name: Configure Pages
        uses: actions/configure-pages@v5

      - name: Build with Jekyll
        run: bundle exec jekyll build --trace
        env:
          JEKYLL_ENV: production

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: _site

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

## 4. Enable GitHub Pages

1. Go to **Settings → Pages**
2. Under **Build and deployment → Source**, select **GitHub Actions**
3. Save

> Token needs `write` access to repo. `repo` scope alone does NOT grant org member write access — must be added as org member in **org Settings → Members**.

## 5. Deploy

```bash
git add -A
git commit -m "Set up Jekyll site + GitHub Pages"
git push origin main
```

Site rebuilds in ~1-2 minutes. Check status:

```bash
gh run list --repo doITbest-ngo/doITbest-ngo.github.io --limit 3
gh run watch <run-id> --repo doITbest-ngo/doITbest-ngo.github.io --exit-status
```

## 6. Adding Content

### New Page

Create `about.md`:

```markdown
---
layout: default
title: About
---

# About doITbest-ngo

Content here.
```

### New Blog Post

Create `_posts/2026-07-11-hello-world.md`:

```markdown
---
layout: default
title: Hello World
---

First post.
```

### Commit & Push

```bash
git add -A
git commit -m "Add about page"
git push origin main
```

## Troubleshooting

| Issue | Fix |
|---|---|
| Push denied | Add `wolkat` as org member with write access |
| `gh repo rename` 404 | Token lacks admin access — rename manually in UI |
| Pages not building | Check Settings → Pages → Source is "GitHub Actions" |
| Site shows 404 | Ensure `index.md` or `README.md` exists at root |
| Build fails | Check Actions tab for error logs |
