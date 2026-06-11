# Feature Landscape

**Domain:** AI-native blog management platform
**Researched:** 2026-06-12

## Table Stakes

Features users expect. Missing = product feels incomplete.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| WYSIWYG Markdown editing | Every blog editor has this (Typora, Notion) | Medium | Milkdown Crepe provides this |
| Git clone/commit/push | Core promise of "git-native" | Medium | isomorphic-git handles this |
| GitHub Pages deploy | Primary deployment target | Low | `hexo deploy` via sidecar |
| Post create/edit/delete | Basic CRUD | Low | File operations + git |
| Image paste/upload | Markdown editing fundamental | Medium | Need image hosting strategy |
| Frontmatter editing | Hexo requires YAML frontmatter | Low | gray-matter parsing + UI |
| Live preview | See rendered output before deploy | Medium | Hexo server via sidecar |
| Tag/category management | Hexo taxonomy system | Low | Read from frontmatter, UI picker |
| Settings UI | API keys, repo config, theme | Low | Zustand + tauri-plugin-store |
| Dark/light mode | Modern app expectation | Low | Tailwind dark mode |
| Post search | Find content in large blogs | Low | SQLite full-text search |

## Differentiators

Features that set product apart. Not expected, but valued.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| AI writing assistant | Generate/rewrite/expand content inline | Medium | Vercel AI SDK streaming |
| Knowledge Base (KB) | Personal wiki that feeds AI context | High | SQLite + embeddings |
| AI-powered post generation from KB | "Write a post about X using my notes" | High | RAG pattern with KB |
| MCP server exposure | External AI agents can manage blog | Medium | @modelcontextprotocol/server |
| Multi-LLM support | Use Claude, GPT, Gemini interchangeably | Low | OpenRouter handles this |
| One-click setup | Clone + configure in single flow | Medium | Guided onboarding wizard |
| Conflict resolution UI | Visual git merge conflict resolver | High | Defer to later phase |
| Batch operations | Publish multiple drafts, bulk tag | Low | UI convenience |
| SEO suggestions | AI-generated meta descriptions, titles | Low | AI prompt template |
| Reading time / word count | Content metrics | Low | Built-in calculation |
| Version history | Visual diff of post changes | Medium | git log + diff display |
| Template system | Reusable post structures | Low | Markdown templates with frontmatter |

## Anti-Features

Features to explicitly NOT build.

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| Real-time collaboration | Scope creep, requires server infrastructure | Single-user local-first design |
| Custom CMS backend | Replicating WordPress/Ghost | Stay git-native, Hexo generates |
| Built-in hosting | Users already have GitHub Pages | Focus on deployment automation |
| Plugin marketplace | Maintenance burden, security risk | Curate essential plugins internally |
| Social media auto-post | Unreliable APIs, scope creep | Provide copy-friendly sharing links |
| Custom domain management | DNS is complex, varies by provider | Link to GitHub Pages docs |
| Comment system | Many existing solutions (Giscus, Utterances) | Document integration in themes |
| Theme editor/builder | Hexo theme ecosystem exists | Support theme switching, not building |
| Multi-blog (initially) | Adds complexity to git/settings | Single repo first, multi later |

## Feature Dependencies

```
WYSIWYG Editor --> Post CRUD --> Git Integration --> Deploy
                                      |
Settings UI --> GitHub Auth --> Git Integration
                                      |
AI Writing --> LLM Config --> Multi-LLM Support
      |
Knowledge Base --> Embeddings --> AI Post Generation
      |
MCP Server --> Post CRUD + KB (both must be stable first)
```

## MVP Recommendation

Prioritize (Phase 1-2):
1. WYSIWYG Markdown editing with frontmatter
2. Git clone + commit + push (single repo)
3. GitHub token authentication
4. Post create/edit/delete
5. Basic settings UI
6. One differentiator: AI writing assistant (inline, streaming)

Defer:
- Knowledge Base: Requires embedding infrastructure (Phase 5)
- MCP Server: Requires stable API surface (Phase 6)
- Conflict resolution: Edge case for single-user (Phase 7+)
- Mobile builds: Desktop-first (Phase 7)
