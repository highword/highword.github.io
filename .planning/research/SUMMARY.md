# Research Summary

**Project:** Open-Blog
**Date:** 2026-06-12
**Sources:** STACK.md, FEATURES.md, ARCHITECTURE.md, PITFALLS.md

## Key Findings

### Stack
- **Frontend:** React 19 + Vite 8 (not Next.js — local-first app, no SSR needed)
- **Desktop:** Tauri 2 (600KB vs Electron's 150MB, mobile support built-in)
- **Git:** isomorphic-git (pure JS, runs in webview) + simple-git fallback
- **AI:** Vercel AI SDK 6 + OpenRouter (one API key → 200+ models)
- **MCP:** @modelcontextprotocol/server (official TypeScript SDK)
- **Editor:** Milkdown Crepe (Typora-like, Markdown-first, free AI feature slot)
- **DB:** SQLite via Drizzle ORM (cache only — git files are source of truth)

### Table Stakes
- WYSIWYG Markdown editor, post CRUD, image upload, frontmatter UI
- Git clone/commit/push (invisible to user), GitHub Pages deploy
- Live preview, search, dark mode, settings UI

### Differentiators (Unique to Open-Blog)
1. **KB System** — Private knowledge base that feeds AI content generation
2. **AI-native writing** — Generate posts from KB, inline AI assist
3. **MCP Server** — AI agents (Claude/Cursor) can manage blog directly
4. **Zero-tech onboarding** — GitHub OAuth → blog live in 5 minutes
5. **Git-native without git knowledge** — Version control for free

### Critical Pitfalls to Avoid
1. **Never show git errors to users** — translate all errors to plain language
2. **AI never auto-publishes** — always human review step
3. **SQLite is cache, not source of truth** — markdown files in git are authoritative
4. **Onboarding must deliver dopamine in 5 minutes** — live blog preview ASAP
5. **Start web-first, add desktop later** — de-risks cross-platform issues
6. **GitHub is deployment target, not runtime dependency** — offline editing must work

### Architecture Decision
```
Tauri 2 app
├── React frontend (webview) with isomorphic-git
├── Rust core (file system, SQLite, security)
├── Node.js sidecar (Hexo CLI only)
└── MCP server (separate process, shared file system)
```

### Suggested Build Order
1. Tauri scaffold + React + routing
2. GitHub auth + repo management
3. Markdown editor (Milkdown Crepe)
4. Post CRUD + frontmatter UI
5. Git pipeline (commit → push → deploy)
6. Onboarding wizard
7. KB system + SQLite indexing
8. AI integration (Vercel AI SDK)
9. MCP Server
10. Desktop packaging + distribution

## Risks & Mitigations

| Top Risk | Likelihood | Mitigation |
|----------|-----------|------------|
| Tauri webview inconsistencies | Medium | Start web-only, add desktop after core works |
| Git conflicts confuse users | Low (single-user) | Visual merge UI as fallback |
| AI cost surprises | Medium | Budget limits, cost preview before generation |
| GitHub OAuth complexity for new users | High | Step-by-step guide with screenshots |
| Hexo Node.js sidecar bundling | Medium | Well-documented Tauri sidecar pattern |

## Confidence Assessment

| Area | Confidence | Notes |
|------|-----------|-------|
| Frontend (React + Vite) | HIGH | Proven, best ecosystem fit |
| Desktop (Tauri 2) | HIGH | Clear winner over Electron for this use case |
| Git (isomorphic-git) | HIGH | Only viable pure-JS option |
| AI (Vercel AI SDK) | HIGH | Best multi-model TypeScript library |
| Editor (Milkdown) | HIGH | Markdown-first, free, feature-rich |
| Architecture | HIGH | Local-first + git-native is well-understood |
| MCP integration | MEDIUM | SDK is pre-1.0 (alpha), but stable enough |
| Mobile (Tauri mobile) | MEDIUM | Tauri 2 supports it, but less battle-tested |
