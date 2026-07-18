# blog.pengyuc.com

Hugo blog with a custom minimal theme that mirrors the design of
[pengyuc.com](https://pengyuc.com) — same tokens (`#FAFAF8` / `#0B0C0E`, mono
chrome + sans prose, 672px column, hairline borders), system-aware dark mode.

## Develop

```bash
hugo server        # http://localhost:1313
hugo --minify      # production build into public/
```

Deploys automatically to GitHub Pages on push to `main`
(`.github/workflows/deploy.yaml`).

## Write

```bash
hugo new content posts/my-post.md
```

Math posts opt into KaTeX by placing `{{</* katex */>}}` once anywhere in the
body; inline math is `\\(...\\)`, display math is `$$...$$`.

## Theme

No external theme — everything lives in this repo:

- `layouts/` — baseof, home (intro + posts by year), page, section, 404,
  partials, and the `katex` shortcode
- `assets/css/main.css` — design tokens and all styling
- `assets/css/chroma.css` — generated syntax palettes
  (`hugo gen chromastyles --style=github` / `github-dark`, dark scoped under
  `html.dark`)

## Syndicate

`/syndicate [post]` (Claude Code skill in `.claude/skills/syndicate/`) drafts
platform-tuned social posts into `social/<slug>.md`:

- Threads — 繁體中文（台灣), ≤500 chars
- X — English, ≤280
- X — 日本語, ≤280 weighted (CJK ×2)

Review, paste, then check off the `Posted` boxes.
