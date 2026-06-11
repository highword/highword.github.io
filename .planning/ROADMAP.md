# Roadmap — Open-Blog v1

## Phase 1: Foundation & Authentication

**Goal:** 搭建应用骨架，用户可以通过 GitHub 登录并连接自己的仓库
**Requirements:** GIT-03, UX-01, UX-02
**UI hint:** yes

### Success Criteria
1. 用户可启动应用并看到完整的界面框架（侧栏、顶栏、主内容区）
2. 用户可通过 GitHub OAuth 或 Token 完成认证，看到自己的 GitHub 用户名
3. 用户可切换深色/浅色模式，界面语言可切换中英文
4. 认证失败时用户看到友好的中文错误提示，而非技术报错

---

## Phase 2: Editor & Post Management

**Goal:** 用户可以在可视化编辑器中创建、编辑和管理博文
**Requirements:** BLOG-01, BLOG-02, BLOG-03, BLOG-04, BLOG-05, BLOG-06, BLOG-07
**UI hint:** yes

### Success Criteria
1. 用户可在所见即所得编辑器中撰写 Markdown 博文，支持粘贴/拖拽图片
2. 用户可通过表单填写标题、标签、分类、日期等 frontmatter 信息
3. 用户可在博文列表中搜索、筛选、查看草稿/已发布状态
4. 用户可实时预览博文的最终渲染效果
5. 用户可将博文设为草稿或发布状态

---

## Phase 3: Git Pipeline & Deployment

**Goal:** 用户一键部署博客到 GitHub Pages，所有 Git 操作在后台静默完成
**Requirements:** GIT-01, GIT-02, GIT-04, GIT-05, GIT-06, BLOG-08, BLOG-09, UX-03
**UI hint:** yes

### Success Criteria
1. 新用户通过 5 步引导向导，5 分钟内完成 GitHub 仓库创建 + Hexo 初始化 + 首篇博文发布上线
2. 用户点击"发布"后博文自动 commit + push + 触发 GitHub Pages 部署，全程无需接触终端
3. 用户可查看博文版本历史，并理解每次修改的变化（非 git log 原始输出）
4. 用户可离线编辑博文，联网后自动同步
5. 用户可通过 GUI 切换博客主题、修改站点配置，无需编辑 YAML 文件

---

## Phase 4: Knowledge Base

**Goal:** 用户可以管理私有知识库，知识库条目与博文建立双向关联
**Requirements:** KB-01, KB-02, KB-03, KB-04
**UI hint:** yes

### Success Criteria
1. 用户可创建、编辑、删除 KB 条目，条目以 Markdown 形式私有存储
2. 用户可为 KB 条目设置标签和分类并按此组织
3. 用户可全文搜索 KB 中的所有内容
4. 用户可看到 KB 条目与博文的双向关联（哪些 KB 生成了哪篇博文，哪篇博文源自哪些 KB）

---

## Phase 5: AI Integration

**Goal:** 用户可通过 AI 辅助写作、从知识库生成博文，并选择自己偏好的模型
**Requirements:** AI-01, AI-02, AI-03, AI-04, AI-05, BLOG-10
**UI hint:** yes

### Success Criteria
1. 用户可在编辑器中呼出 AI 辅助：补全、扩写、改写选中内容
2. 用户可选择多个 KB 条目，AI 整合生成完整博文草稿供审阅
3. AI 可自动建议博文标题和标签，用户一键采纳或修改
4. 用户可在设置中切换不同 LLM 模型（Claude/GPT/Gemini/本地模型）
5. 博文支持 Markdown 原始模式和 AI 生成的增强 HTML 模式，读者可切换查看

---

## Phase 6: MCP Server & Packaging

**Goal:** 外部 AI Agent 可通过 MCP 协议操作博客，应用可作为桌面软件安装使用
**Requirements:** MCP-01
**UI hint:** no

### Success Criteria
1. Claude Code / Cursor 等 AI Agent 可通过 MCP 协议创建、编辑、发布博文和 KB 条目
2. 用户可在 AI Agent 环境中通过自然语言对话完成博文发布全流程
3. 应用可打包为 Windows/macOS 安装包，双击安装即可使用

---

## Traceability Matrix

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
*Created: 2026-06-12*
