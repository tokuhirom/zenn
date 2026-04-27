# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a content repository for [Zenn](https://zenn.dev), a Japanese tech-publishing platform. There is no application code — the repo holds Markdown articles, multi-chapter books, and their images, managed via `zenn-cli`. Most articles are published under the `sakura_internet` publication. Content is written in Japanese.

## Common commands

```bash
npx zenn preview         # Local preview server (http://localhost:8000 by default)
npx zenn new:article     # Scaffold a new article in articles/
npx zenn new:book        # Scaffold a new book directory in books/
npx zenn list:articles
npx zenn list:books
```

There are no build, lint, or test steps — Zenn renders the Markdown server-side after push.

## Layout

- `articles/<slug>.md` — single-file articles. Filename slug is the URL slug.
- `books/<slug>/` — one directory per book containing `config.yaml` (chapter order, metadata), `cover.jpg`, and one Markdown file per chapter. Chapter order is controlled by the `chapters:` list in `config.yaml`, not by filename.
- `images/` — shared image assets. Per-book images live under `images/<book-slug>/`. Reference them with absolute repo paths (e.g. `/images/foo.png`) so they resolve in both preview and published views.

## Frontmatter

Every article and chapter starts with YAML frontmatter. Required keys for articles:

```yaml
---
title: "..."
emoji: "📚"           # single emoji shown as the article icon
type: "tech"          # "tech" or "idea"
topics: ["..."]       # tag list
publication_name: "sakura_internet"   # omit for personal posts
published: true       # set false to keep as draft
---
```

Books use `config.yaml` instead (see existing books for the schema). Flip `published: false` → `true` only when the post is ready to go live; the commit that flips it is effectively the publish action.

## Workflow notes

- Drafts (`published: false`) are safe to commit; they won't appear publicly.
- Slugs (filenames) are part of the public URL once published — don't rename after publishing.
- When adding images, place them under `images/` and link with absolute paths from the repo root.
