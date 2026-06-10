# Technology Stack

## Core Framework
- **Hexo** v7.0.0 — Static site generator (Node.js-based)
- **Node.js** — Runtime environment

## Rendering Engines
- `hexo-renderer-ejs` ^2.0.0 — EJS template rendering
- `hexo-renderer-marked` ^6.0.0 — Markdown to HTML
- `hexo-renderer-pug` ^3.0.0 — Pug template rendering (required by Butterfly theme)
- `hexo-renderer-stylus` ^3.0.0 — Stylus CSS preprocessing

## Content Generators
- `hexo-generator-index` ^3.0.0 — Homepage generation
- `hexo-generator-archive` ^2.0.0 — Archive pages
- `hexo-generator-category` ^2.0.0 — Category pages
- `hexo-generator-tag` ^2.0.0 — Tag pages

## Deployment
- `hexo-deployer-git` ^4.0.0 — Git-based deployment to GitHub Pages
- Target: `https://github.com/giyanwei/giyanwei.github.io.git` (main branch)

## Theme
- **Butterfly** — Feature-rich Hexo theme (installed as git submodule in `themes/butterfly/`)
- Template language: Pug
- Styling: Stylus

## Development Tools
- `hexo-server` ^3.0.0 — Local development server
- GitHub Dependabot — Daily npm dependency updates

## Social/Third-party
- `social-share.js` ^1.0.16 — Social sharing buttons

## Build Scripts
| Command | Action |
|---------|--------|
| `npm run build` | `hexo generate` — Build static files |
| `npm run clean` | `hexo clean` — Remove generated files |
| `npm run deploy` | `hexo deploy` — Deploy to GitHub Pages |
| `npm run server` | `hexo server` — Local dev server |
