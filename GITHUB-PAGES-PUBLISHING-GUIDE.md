# GitHub Pages Publishing Guide

Lifecycle: active
Role: guide
Project: agent-playbook-suite
Updated: 2026-05-25

This guide describes how to turn this folder into the public `agent-playbook-suite` GitHub repository and publish a GitHub Pages site from it, while keeping the Markdown files in the repository as public technical reference material.

## Goal

Publish a public GitHub Pages site for the agent playbook suite.

The repository should serve two purposes:

1. Public website: a small GitHub Pages site with a landing page and a blog page.
2. Technical reference repository: the Markdown files in this folder remain public and linkable from the website.

The website should not replace the Markdown references. It should introduce the project, link to the blog post, and point readers to the source/reference documents in the repo.

## Recommended Shape

Keep the existing Markdown files at the repository root as technical reference material, then add a separate `site/` directory for the rendered Pages site.

Proposed final structure:

```text
agent-playbook-suite/
  .docs.toml
  INDEX.md
  overview.md
  blog-post.md
  briefing.md
  hn-technical-writing-style-guide.md
  AGENTS.md
  CLAUDE.md
  GITHUB-PAGES-PUBLISHING-GUIDE.md

  site/
    Gemfile
    _config.yml
    index.md
    blog.md
    references.md

  .github/
    workflows/
      pages.yml
```

Why use `site/` instead of publishing from the repository root:

- The root Markdown files can remain docs-cli-managed technical records.
- Jekyll pages need YAML front matter at the very top of rendered pages, which conflicts with docs-cli's convention that managed files start with an H1 followed by metadata.
- A GitHub Actions Pages workflow can build from any directory, so `site/` can be the public website source while the root stays the technical reference corpus.

## Step 1: Adopt The Folder With docs-cli

Before creating the GitHub repository, make the Markdown tree explicit and validated.

Run a dry-run migration first:

```sh
cd agent-playbook-suite
docs migrate . --json > /tmp/agent-playbook-suite-migrate.json
```

Review the migration plan:

```sh
less /tmp/agent-playbook-suite-migrate.json
```

If the plan looks right, apply it:

```sh
docs migrate . --apply
```

Then validate and generate the index:

```sh
docs check .
docs index --root .
```

Expected result:

- Every public Markdown file has docs-cli metadata.
- `docs check .` exits cleanly.
- `INDEX.md` is generated and can serve as the technical reference index for the repository.

Important review point: the user has decided that all Markdown files in this folder can be public. Still, before pushing to GitHub, do one final human read-through of the migrated files to make sure no private tokens, private customer names, private repo paths, or unpublished credentials are present.

## Step 2: Create The GitHub Repository

Initialize git if this folder is not already a repository:

```sh
cd agent-playbook-suite
git init
git branch -M main
git status --short
```

Commit the docs-cli-managed reference tree:

```sh
git add .
git commit -m "Initial public agent playbook suite docs"
```

Create the public GitHub repository and push it:

```sh
gh repo create ArtRichards/agent-playbook-suite \
  --public \
  --source=. \
  --remote=origin \
  --push
```

After this, the repository URL should be:

```text
https://github.com/ArtRichards/agent-playbook-suite
```

## Step 3: Add The Pages Site Source

Add a `site/` directory. This is the rendered website source, separate from the technical reference Markdown at the repository root.

### `site/_config.yml`

```yaml
title: Agent Playbook Suite
description: A file-backed workflow for resumable agent-led software projects.
url: "https://artrichards.github.io"
baseurl: "/agent-playbook-suite"
theme: minima

header_pages:
  - blog.md
  - references.md
```

### `site/Gemfile`

```ruby
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins
```

### `site/index.md`

Use this as the public landing page. It should be a concise rewrite of `overview.md`, not the full technical reference.

```markdown
---
layout: home
title: Agent Playbook Suite
---

# Agent Playbook Suite

A five-skill workflow for making Claude Code projects resumable by keeping project state on disk instead of in chat.

Start with the blog post:

- [State lives on disk, not in chat]({{ site.baseurl }}/blog/)

Technical references are available in the source repository:

- [Repository](https://github.com/ArtRichards/agent-playbook-suite)
- [Overview](https://github.com/ArtRichards/agent-playbook-suite/blob/main/overview.md)
- [Reference index](https://github.com/ArtRichards/agent-playbook-suite/blob/main/INDEX.md)
```

### `site/blog.md`

Use this page for the blog post. Copy the current `blog-post.md` body into this file under Jekyll front matter.

```markdown
---
layout: page
title: State lives on disk, not in chat
permalink: /blog/
---

<copy blog-post.md body here>
```

### `site/references.md`

Use this page to point readers to the public Markdown reference files in the repository.

```markdown
---
layout: page
title: Technical References
permalink: /references/
---

# Technical References

The source repository contains the full Markdown reference set:

- [Overview](https://github.com/ArtRichards/agent-playbook-suite/blob/main/overview.md)
- [Blog post source](https://github.com/ArtRichards/agent-playbook-suite/blob/main/blog-post.md)
- [Briefing](https://github.com/ArtRichards/agent-playbook-suite/blob/main/briefing.md)
- [Technical writing style guide](https://github.com/ArtRichards/agent-playbook-suite/blob/main/hn-technical-writing-style-guide.md)
- [Repository docs index](https://github.com/ArtRichards/agent-playbook-suite/blob/main/INDEX.md)
```

## Step 4: Add GitHub Pages Workflow

Use GitHub Actions rather than branch-folder publishing. This avoids using `/docs` as the Pages source and keeps the docs-cli-managed Markdown tree separate from the rendered website.

Create `.github/workflows/pages.yml`:

```yaml
name: Deploy Pages

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
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: "3.3"
          bundler-cache: true
          working-directory: site
      - run: bundle exec jekyll build
        working-directory: site
      - uses: actions/upload-pages-artifact@v3
        with:
          path: site/_site

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - id: deployment
        uses: actions/deploy-pages@v4
```

Commit and push:

```sh
git add site .github/workflows/pages.yml
git commit -m "Add GitHub Pages site"
git push
```

## Step 5: Enable Pages In GitHub

In the GitHub repository:

1. Open Settings.
2. Open Pages.
3. Under Build and deployment, choose Source: GitHub Actions.
4. Save if needed.
5. Open the Actions tab and wait for the Pages workflow to complete.

Expected public URL:

```text
https://artrichards.github.io/agent-playbook-suite/
```

Expected blog URL:

```text
https://artrichards.github.io/agent-playbook-suite/blog/
```

## Step 6: Verification Checklist

After deployment:

```sh
curl -I https://artrichards.github.io/agent-playbook-suite/
curl -I https://artrichards.github.io/agent-playbook-suite/blog/
```

Then verify manually:

- Landing page loads.
- Blog page loads.
- Repository links work.
- Reference links point to the GitHub repository.
- No internal-only warnings or private notes appear on the public Pages site unless intentionally published.
- `docs check .` still exits cleanly from the repository root.
- `docs index --root . --dry-run` is idempotent.

## Step 7: Ongoing Publishing Flow

For future edits:

1. Edit the root Markdown references or `site/` pages.
2. Run:
   ```sh
   docs check .
   docs index --root .
   ```
3. If `site/blog.md` is copied from `blog-post.md`, keep them in sync manually, or add a later script to generate the site page from the source file.
4. Commit and push.
5. GitHub Actions redeploys Pages.

## Notes And Tradeoffs

This plan intentionally keeps two layers:

- Root Markdown: public technical reference, docs-cli-managed.
- `site/`: small public website, Jekyll-managed.

The duplication between `blog-post.md` and `site/blog.md` is acceptable for the first version. If that becomes annoying, add a small build script later that reads `blog-post.md`, wraps it with Jekyll front matter, and writes `site/blog.md` during the Pages build.

Avoid publishing directly from `/docs` because this repository should not need a `/docs` source folder just to satisfy GitHub Pages branch publishing. GitHub Actions is the better fit here.

## Official References

- GitHub Pages publishing source: https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
- GitHub Pages custom workflows: https://docs.github.com/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages
- GitHub Pages Jekyll content: https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/adding-content-to-your-github-pages-site-using-jekyll
