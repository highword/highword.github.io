# Project Structure

```
Gu-Galaxy/
├── _config.yml                 # Main Hexo configuration
├── _config.landscape.yml       # Landscape theme config (unused, bundled default)
├── package.json                # Node.js dependencies
├── .gitignore                  # Excludes node_modules/, public/, .deploy*/
├── .github/
│   └── dependabot.yml          # Automated dependency updates
├── scaffolds/                  # Templates for content creation
│   ├── draft.md
│   ├── page.md
│   └── post.md
├── source/                     # Content source directory
│   ├── _posts/                 # Blog posts (22 articles)
│   │   ├── Have-fun-in-Thailand.md
│   │   ├── How-to-use-ChatGPT.md
│   │   ├── Reflections-on-GPT-and-the-Future.md
│   │   ├── Devops---基于Docker实现GitLab企业开发过程全自动化.md
│   │   ├── k8s初体验-*.md (3 articles)
│   │   ├── Python*.md (3 articles)
│   │   ├── HTTPs*.md (2 articles)
│   │   └── ... (other tech/life articles)
│   └── Categories/
│       └── index.md            # Categories page
├── themes/
│   ├── .gitkeep
│   └── butterfly/              # Active theme (git submodule)
│       ├── _config.yml         # Theme configuration
│       ├── layout/             # Pug templates
│       ├── source/             # Theme assets (CSS, JS)
│       └── languages/          # i18n files
└── node_modules/               # Dependencies (gitignored in production)
```

## Content Categories (by topic)
- **DevOps/Cloud**: Docker, k8s, FTP, frp, GitLab CI/CD
- **Web/Security**: OAuth, HTTPS, 证书验证
- **Programming**: Python basics/OOP/advanced
- **AI/LLM**: ChatGPT usage, GPT reflections
- **Life/Travel**: Thailand trip
- **Meta**: 博客搭建, 图床配置, Hexo教程
- **Database**: 索引深度全解
