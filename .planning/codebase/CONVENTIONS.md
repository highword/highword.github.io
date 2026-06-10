# Conventions

## Post Naming
- English titles: Kebab-case (`How-to-use-ChatGPT.md`, `Have-fun-in-Thailand.md`)
- Chinese titles: Direct Chinese characters (`frp内网穿透教程.md`, `k8s初体验-minikube突破隔断访问限制.md`)
- Mixed: Prefix in English/pinyin + Chinese description

## Front Matter
Standard Hexo YAML front matter expected:
```yaml
---
title: Post Title
date: YYYY-MM-DD HH:mm:ss
tags: [tag1, tag2]
categories: [category]
---
```

## Language
- Blog posts: Mix of English and Chinese
- Site language setting: `en`
- Author name: "Quasar Gu"

## Theme Configuration
- Highlight theme: `light`
- Social icons: GitHub + Email only
- Navigation: Home, Archives, Tags (Categories commented out)
- Code blocks: Copy enabled, language shown, no shrink

## Git Workflow
- Single `main` branch
- Direct commits (no PR workflow for content)
- Dependabot for automated dependency PRs
- `node_modules/` tracked in repo (unusual — normally gitignored)

## Deployment
- Manual `hexo deploy` command
- Deploys to separate `giyanwei.github.io` repository
