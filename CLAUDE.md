# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal academic website for **Feiyue Xia** (Timothy Xia), a Petroleum Engineering PhD student at UT Austin. It is a **Hugo** static site built on the **HugoBlox "Academic CV"** template, pulled in entirely as **Hugo Modules** (no vendored theme — see `go.mod` / `config/_default/module.yaml`). Styling is **Tailwind CSS v4** (driven by `build.writeStats: true` → `hugo_stats.json`), with Preact and Pagefind search.

## Commands

```powershell
hugo server --port 1313 --disableFastRender   # dev server at http://localhost:1313 (matches .claude/launch.json)
hugo --gc --minify                            # production build into public/ — the canonical "is it broken?" check
hugo mod get -u                               # update theme modules
```

- There is **no test suite, linter, or package.json**. Verification = a clean `hugo --gc --minify` (exit 0, watch for `WARN`s) plus checking rendered output.
- **`public/` is not purged between builds.** When deleting or renaming content, `Remove-Item -Recurse -Force public` before rebuilding, or stale pages/typo'd folders linger.
- In this session, screenshots via the preview tools time out (Pagefind 404 + livereload websocket prevent network-idle). Verify with `preview_eval` DOM inspection or by grepping the built HTML in `public/` instead.
- **Minified HTML uses unquoted attributes** (`src=/media/...`), so regex checks against `public/` must not assume quotes.

## Architecture

### Content is data, not layout
Pages are assembled from **blocks** declared in front matter `sections:`, not hand-written HTML. The key landing pages:
- `content/_index.md` — homepage: `resume-biography-3` → `collection` (Selected Publications, `featured_only`) → `collection` (Featured Projects) → `collection` (Recent Posts).
- `content/experience.md` — `resume-experience`, `resume-skills`, `resume-awards` (×2), `resume-languages`.
- `content/publications/_index.md` — two `collection` blocks split by `publication_type` (`article-journal` vs `paper-conference`), `view: citation`.
- `content/projects/_index.md`, `content/blog/_index.md` — list pages.

### Profile data lives in `data/authors/`
The resume blocks read a YAML profile selected by `username:` (the file stem). `data/authors/me.yaml` is the owner (`is_owner: true`) and drives the bio, education, experience, skills, awards, languages, and social links.

**Non-obvious:** the `resume-awards` block only reads the **`awards:`** key. To render a second "Academic Activities & Service" section, the service items are stored under an `awards:` key in a separate `data/authors/service.yaml`, and `experience.md` points the second `resume-awards` block at `username: service`. Don't "fix" this by renaming the key.

### Theme color engine
`config/_default/params.yaml` → `hugoblox.theme.colors.primary` is set to `#BF5700` (UT burnt orange). A 7-char hex or a named Tailwind palette both work — the theme auto-generates the full 50–950 scale and emits CSS vars like `--hb-primary-500-rgb: 191 87 0`. Theme config is `partialCached`, so **restart `hugo server` after changing colors.**

### Overriding theme module files
Module templates resolve from the cache (`...\hugo_cache\modules\...\blox@<ver>\layouts\...`). To customize one, copy it to the **same relative path under the project's `layouts/`** and it shadows the module version. Current project overrides (each exists for a specific reason — preserve the rationale comments):
- `layouts/_partials/hbx/blocks/resume-experience/block.html` — adds per-entry `logo` support (extracts a `logo` key the stock block drops, renders it as the timeline marker image). Used by experience + education logos.
- `layouts/_partials/hbx/blocks/resume-biography-3/block.html` — same `logo` support for the homepage education badges.
- `layouts/_partials/page_metadata_authors.html` — renders co-authors as plain text when they have no profile page (stock block emitted empty `<a href="">`); bolds the owner.
- `layouts/_partials/views/article-grid--start.html` — caps grid columns at item count so a lone card (e.g. one blog post) isn't stretched into a narrow column.

**Gotcha:** files copied from the module cache via PowerShell come in **read-only**; clear it before editing (`Set-ItemProperty <file> -Name IsReadOnly -Value $false`) and `Read` the file before `Edit`.

### Logo / favicon assets
- Header logo ← `assets/media/logo.png`.
- Browser-tab favicon + apple-touch + share image ← `assets/media/icon.png` (function `get_site_icon` prefers `media/icon.svg` → `icon.png` → `logo.png`; the theme ships a default `icon.png`, so a project `icon.png` is required to override it).
- Both are copies of `assets/media/icons/website_logo.png`. School/org logos for resume entries live in `assets/media/icons/` (`UT.svg`, `UPC_logo.jpg`, `HAL_logo.jpeg`) and are referenced by filename in the profile YAML.

## Content authoring conventions

**Publications** — one folder per paper: `content/publications/<slug>/index.md`. Front matter: `title`, `authors` (list; use the literal string `me` for the owner so it links to the profile, plain strings like `X. Tang` for co-authors), `date` + `publishDate` as `YYYY-MM-DDT00:00:00Z`, `publication_types: ["article-journal"]` or `["paper-conference"]`, `publication`, optional `featured: true` (homepage), and `hugoblox.ids.doi` for the DOI link.

**Future-dated content is hidden by default.** Hugo drops pages whose `publishDate` is in the future. A paper with a future `date` (e.g. an accepted-but-unpublished 2026 item) must be given a past `publishDate` to appear.

**Projects** — `content/projects/<slug>/index.md`: `title`, `date`, `summary`, `tags`, optional `links`.

**Blog** — placeholder post at `content/blog/welcome/`. The user does not actively blog; don't fabricate posts.

The site is bilingual-aware (`content/authors/_index.md` cascades `render: never` on the authors taxonomy). All theme demo content (Courses/Events/Slides, sample papers/projects/posts) has been removed and should stay removed.

## Workflow note

Nothing is committed automatically — always ask before `git commit`/`push`. The site has no CI deploy configured in-repo beyond the Netlify integration module.
