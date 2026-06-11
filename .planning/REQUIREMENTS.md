# Requirements — Open-Blog v1

## Blog Management

- [ ] **BLOG-01**: 用户可通过 WYSIWYG 编辑器（Milkdown Crepe）创建和编辑博文
- [ ] **BLOG-02**: 用户可创建、编辑、删除博文
- [ ] **BLOG-03**: 用户可通过粘贴或拖拽上传图片到博文中
- [ ] **BLOG-04**: 用户可通过表单界面编辑 frontmatter（标题、标签、分类、日期）
- [ ] **BLOG-05**: 用户可将博文设为草稿或已发布状态
- [ ] **BLOG-06**: 用户可搜索和筛选博文列表
- [ ] **BLOG-07**: 用户可实时预览博文渲染效果
- [ ] **BLOG-08**: 用户可查看博文的版本历史（基于 git log）
- [ ] **BLOG-09**: 用户可批量发布草稿或批量修改标签
- [ ] **BLOG-10**: 博文支持双模式：Markdown 原始模式 + AI 生成的增强 HTML 模式，读者可切换

## Git & Deployment

- [ ] **GIT-01**: 所有内容操作底层通过 Git 完成，用户无需了解 git
- [ ] **GIT-02**: 用户可一键部署博客到 GitHub Pages
- [ ] **GIT-03**: 用户可通过 GitHub token 或 OAuth 认证
- [ ] **GIT-04**: 新用户可通过引导流程一键创建 GitHub repo + Hexo 项目 + 部署上线
- [ ] **GIT-05**: 用户可通过 GUI 选择主题并可视化修改博客配置（不编辑 YAML）
- [ ] **GIT-06**: 用户可离线编辑，联网后自动同步推送

## Knowledge Base

- [ ] **KB-01**: 用户可创建、编辑、删除 KB 条目（私有 Markdown 文件）
- [ ] **KB-02**: 用户可为 KB 条目添加标签和分类
- [ ] **KB-03**: 用户可全文搜索 KB 内容
- [ ] **KB-04**: KB 条目与博文可双向关联，可追溯哪些 KB 生成了哪篇博文

## AI

- [ ] **AI-01**: 用户可在编辑器内使用 AI 补全、扩写、改写内容
- [ ] **AI-02**: 用户可选择多个 KB 条目，由 AI 整合生成一篇博文
- [ ] **AI-03**: AI 可根据博文内容自动建议标题和标签
- [ ] **AI-04**: 用户可选择不同 LLM 模型（Claude、GPT、Gemini、本地模型）
- [ ] **AI-05**: AI 可根据 Markdown 原料生成增强 HTML 版本（更好的排版、样式、交互组件）

## MCP & Integration

- [ ] **MCP-01**: 提供 MCP Server，外部 AI Agent（Claude Code/Cursor）可创建/发布/查询博文和 KB

## UX

- [ ] **UX-01**: 支持深色/浅色模式切换
- [ ] **UX-02**: 支持多语言界面（中文/英文）
- [ ] **UX-03**: 新用户引导系统：5 步向导，非技术用户 5 分钟内发布第一篇博文

---

## v2 (Deferred)

- 博文模板系统
- 图表/关系图可视化（KB 连接图）
- 从 Notion/Obsidian 导入
- SEO 优化建议
- 移动端 App（Tauri mobile）

## Out of Scope

- 实时多人协作 — MVP 聚焦单用户
- 自建托管 — 只支持 GitHub Pages
- 评论系统 — 使用现有方案（Giscus）
- 自定义主题开发 — 支持切换主题，不支持从零创建
- 社交媒体自动发帖 — 范围过大

---

## Traceability

| REQ-ID | Phase | Status |
|--------|-------|--------|
| GIT-03 | 1 | Pending |
| UX-01 | 1 | Pending |
| UX-02 | 1 | Pending |
| BLOG-01 | 2 | Pending |
| BLOG-02 | 2 | Pending |
| BLOG-03 | 2 | Pending |
| BLOG-04 | 2 | Pending |
| BLOG-05 | 2 | Pending |
| BLOG-06 | 2 | Pending |
| BLOG-07 | 2 | Pending |
| GIT-01 | 3 | Pending |
| GIT-02 | 3 | Pending |
| GIT-04 | 3 | Pending |
| GIT-05 | 3 | Pending |
| GIT-06 | 3 | Pending |
| BLOG-08 | 3 | Pending |
| BLOG-09 | 3 | Pending |
| UX-03 | 3 | Pending |
| KB-01 | 4 | Pending |
| KB-02 | 4 | Pending |
| KB-03 | 4 | Pending |
| KB-04 | 4 | Pending |
| AI-01 | 5 | Pending |
| AI-02 | 5 | Pending |
| AI-03 | 5 | Pending |
| AI-04 | 5 | Pending |
| AI-05 | 5 | Pending |
| BLOG-10 | 5 | Pending |
| MCP-01 | 6 | Pending |

---
*Generated: 2026-06-12*
