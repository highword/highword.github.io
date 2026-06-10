# Open-Blog

## What This Is

一个面向非技术用户的 AI-native 博客管理平台。用户无需了解 Git、Hexo、命令行，通过可视化界面即可一键搭建 GitHub Pages 博客、管理知识库（KB）和博文、借助 AI 生成内容。底层 git-native，三端覆盖（Web → 桌面 → 移动端）。

## Core Value

非技术用户在 5 分钟内零门槛创建并发布第一篇博文——完全不需要知道 Git/Hexo/终端的存在。

## Requirements

### Validated

- ✓ Hexo + Butterfly 主题博客搭建与 GitHub Pages 部署 — existing (Gu-Galaxy)

### Active

- [ ] 本地 Web 管理后台（博文 CRUD、预览、发布）
- [ ] Git-native：所有操作底层通过 Git 完成，用户无感
- [ ] 一键初始化：新用户通过 GUI 自动创建 GitHub repo + Hexo 项目 + 部署
- [ ] 主题选择与可视化配置（不需要编辑 YAML）
- [ ] KB 系统：私有知识库，可选择性公开为博文
- [ ] AI 生成系统：从上下文/KB 通过 AI 生成博文或 KB 条目
- [ ] KB → 博文转化：一个或多个 KB 经 AI 整合生成博文，双向关联
- [ ] MCP Server：嵌入 Claude Code/Cursor 等 AI Agent 环境，傻瓜式生成+推送博文
- [ ] 多模型支持：用户可选择不同 LLM（Claude、GPT、本地模型等）
- [ ] 新手引导系统：step-by-step onboarding，非技术用户友好

### Out of Scope

- 自建博客服务器托管 — 只支持 GitHub Pages 静态部署（MVP）
- 多人协作/团队博客 — MVP 聚焦个人用户
- 付费/商业化功能 — 开源优先
- 移动端 App — MVP 后期迭代

## Context

- 当前 Gu-Galaxy 博客作为 dogfooding 对象，第一个接入 Open-Blog 管理的实例
- 面向非技术用户意味着：所有 Git 操作、Hexo 命令、YAML 配置、部署流程必须完全封装
- AI-native 核心理念：KB 是 source，博文是 product，AI Agent 在中间做转化
- MCP 集成允许用户在编程环境中（Claude Code/Cursor）通过对话生成内容并一条龙推送到博客
- LLM-Wiki 思想：快速从上下文中提取知识存入 KB
- 产品形态路线：本地 Web App → 桌面应用 → 移动端

## Constraints

- **Tech stack**: 待研究确定（需支持 Web + 桌面封装 + 未来移动端）
- **部署目标**: GitHub Pages（静态站点）
- **博客引擎**: Hexo（MVP），未来可扩展支持其他静态生成器
- **开源**: 项目本身开源，可推广的普适性工具
- **用户门槛**: 零技术门槛是硬性约束，不是 nice-to-have

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| Git-native 架构 | 所有内容存储和版本管理基于 Git，确保数据主权在用户手中 | — Pending |
| KB-博文双向关联 | KB 是原始知识，博文是加工产物，AI 做桥梁 | — Pending |
| 先 Web 后桌面后移动 | Web 最快验证，桌面用 Tauri/Electron 封装，移动端最后 | — Pending |
| MCP Server 集成 | 让 AI Agent 环境直接操作博客，符合 AI-native 理念 | — Pending |
| 多模型支持 | 不绑定单一 LLM 供应商，用户可选 | — Pending |
| 产品名: Open-Blog | 暂定，后续讨论 | — Pending |

## Evolution

This document evolves at phase transitions and milestone boundaries.

**After each phase transition** (via `/gsd-transition`):
1. Requirements invalidated? → Move to Out of Scope with reason
2. Requirements validated? → Move to Validated with phase reference
3. New requirements emerged? → Add to Active
4. Decisions to log? → Add to Key Decisions
5. "What This Is" still accurate? → Update if drifted

**After each milestone** (via `/gsd-complete-milestone`):
1. Full review of all sections
2. Core Value check — still the right priority?
3. Audit Out of Scope — reasons still valid?
4. Update Context with current state

---
*Last updated: 2026-06-11 after initialization*
