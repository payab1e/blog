# Blog Guidelines for Claude

## Stack
- Jekyll + Chirpy theme (v7.5) hosted on GitHub Pages via custom Actions workflow
- Posts go in `_posts/`, filename format: `YYYY-MM-DD-title-slug.md`
- Deployment: push to `main` → GitHub Actions builds → deploys to `blog.payable.kr`

## Writing Posts

### Front Matter
```yaml
---
title: "Post Title Here"
date: YYYY-MM-DD HH:MM:SS +0900
categories: [category1, category2]
tags: [tag1, tag2, tag3]
---
```

### Date Rules
- **Always use a past date/time.** Jekyll excludes posts with future timestamps from builds.
- `_config.yml` has `future: true` as a safeguard, but set the date correctly regardless.
- Use `+0900` (KST) for timezone. Set time to `00:00:00` unless a specific time matters.
- Example: if today is 2026-04-05, use `date: 2026-04-04 00:00:00 +0900`

### Categories and Tags
- Use **lowercase** for both categories and tags. The theme uses Jekyll's `slugify` filter to build links (e.g., `CTF` → links to `/categories/ctf/`), while jekyll-archives generates pages at `:name`. Using lowercase avoids case-mismatch on the case-sensitive Linux build server.
- Tags with hyphens are fine: `blind-sql-injection`, `ctf-writeup`

### Fake/Example URLs in Posts
- URLs in fenced code blocks or backtick spans are safe — they render as `<code>`, not links, so htmlproofer ignores them.
- Do not put bare (unquoted, non-code) fake URLs in prose — they may be auto-linked by kramdown.

## Build & CI Notes
- htmlproofer v5 (`html-proofer ~> 5.0`) runs as the "Test site" step in the Actions workflow.
- Current config: `--disable-external` (skips external URL checks) and `|| true` (does not fail the build on htmlproofer errors).
- If htmlproofer errors appear in logs, check the "Test site" step output in GitHub Actions for the specific broken link.

## Site Structure
- `_tabs/` — sidebar nav pages. Currently only `about.md` and `archives.md` (categories/tags tabs removed intentionally).
- `_config.yml` key settings: `theme_mode: dark`, `future: true`, timezone `Asia/Seoul`
- Favicons: all required files exist in `assets/img/favicons/`
