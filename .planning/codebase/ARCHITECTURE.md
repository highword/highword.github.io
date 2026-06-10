# Architecture

## System Type
Static blog site — content authored in Markdown, built to HTML via Hexo, deployed to GitHub Pages.

## Build Pipeline
```
source/_posts/*.md  →  Hexo Generate  →  public/  →  hexo deploy  →  GitHub Pages
```

## Content Model
- **Posts** (`source/_posts/`) — Blog articles in Markdown with YAML front matter
- **Pages** (`source/Categories/`) — Static pages (categories listing)
- **Scaffolds** (`scaffolds/`) — Templates for new posts/pages/drafts
- **Assets** — Images referenced from posts (external hosting or local)

## Theme Architecture
Butterfly theme (`themes/butterfly/`) provides:
- Pug layouts (`layout/`) — Page structure and components
- Stylus styles (`source/css/`) — Styling
- JavaScript (`source/js/`) — Client-side interactivity
- i18n (`languages/`) — Multilingual support (en, zh-CN, zh-TW)

## Configuration Hierarchy
1. `_config.yml` — Main Hexo config (site metadata, URL, deployment)
2. `themes/butterfly/_config.yml` — Theme-specific settings (nav, social, images)
3. Individual post front matter — Per-post overrides

## Deployment Architecture
```
Local Authoring → hexo generate → hexo deploy (git push) → GitHub Pages CDN
```
No server-side processing. All content is pre-rendered at build time.
