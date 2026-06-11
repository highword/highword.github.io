# Architecture Research

**Domain:** AI-native, git-native blog management platform
**Researched:** 2026-06-12

## System Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Tauri Desktop App                        │
├─────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────┐  │
│  │              React Frontend (Webview)                   │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌─────────┐ │  │
│  │  │ Editor   │ │ KB UI    │ │ Settings │ │Onboard  │ │  │
│  │  │(Milkdown)│ │          │ │          │ │ Wizard  │ │  │
│  │  └──────────┘ └──────────┘ └──────────┘ └─────────┘ │  │
│  │  ┌──────────────────────────────────────────────────┐ │  │
│  │  │        isomorphic-git (in-webview git ops)        │ │  │
│  │  └──────────────────────────────────────────────────┘ │  │
│  └───────────────────────────────────────────────────────┘  │
│                          ↕ IPC                               │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Tauri Rust Core                            │  │
│  │  • File system access (plugin-fs)                      │  │
│  │  • SQLite (rusqlite / plugin-sql)                      │  │
│  │  • Shell commands (plugin-shell)                       │  │
│  │  • Secure credential storage                          │  │
│  │  • Window management                                  │  │
│  └───────────────────────────────────────────────────────┘  │
│                          ↕ Sidecar                           │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Node.js Sidecar                            │  │
│  │  • Hexo CLI (generate, server, deploy)                │  │
│  │  • simple-git (fallback for complex ops)              │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                          ↕ HTTP
┌─────────────────────────────────────────────────────────────┐
│                     External Services                         │
│  • GitHub API (OAuth, repo creation, Pages)                  │
│  • LLM APIs (OpenRouter / direct providers)                  │
│  • MCP Clients (Claude Desktop, Cursor, etc.)                │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  MCP Server (separate process)                │
│  • stdio transport (local AI agents)                         │
│  • Tools: create-post, create-kb, deploy, search             │
│  • Communicates with main app via local API/IPC              │
└─────────────────────────────────────────────────────────────┘
```

## Component Boundaries

| Component | Responsibility | Interface |
|-----------|---------------|-----------|
| React Frontend | UI, editor, user interaction | Tauri IPC commands |
| isomorphic-git | Git clone/commit/push in webview | JS API (fs adapter) |
| Tauri Rust Core | File system, SQLite, credentials, shell | IPC commands/events |
| Node.js Sidecar | Hexo CLI operations | Shell plugin stdin/stdout |
| AI Service | LLM calls via Vercel AI SDK | HTTP to provider APIs |
| MCP Server | External AI agent interface | stdio / HTTP transport |
| GitHub Service | OAuth, repo CRUD, Pages config | GitHub REST/GraphQL API |

## Data Model

### Content (Git-tracked, Markdown files)

```
blog-repo/
├── source/
│   ├── _posts/          # Published blog posts
│   │   └── my-post.md   # Hexo post (YAML frontmatter + content)
│   └── _kb/             # Knowledge Base entries (private dir)
│       └── topic-x.md   # KB entry (custom frontmatter + content)
├── _config.yml          # Hexo config
└── themes/              # Hexo theme
```

### Post frontmatter (extended)
```yaml
---
title: My Post
date: 2026-06-12
tags: [python, tutorial]
categories: [Development]
kb_sources: [kb-123, kb-456]  # Links to KB entries that generated this
---
```

### KB entry frontmatter
```yaml
---
id: kb-123
title: Python decorators
created: 2026-06-12
updated: 2026-06-12
tags: [python, advanced]
linked_posts: [my-post]  # Posts generated from this KB
public: false            # Not deployed to blog
---
```

### Metadata (SQLite, local only)

```sql
-- Fast search and relationship queries
posts(id, title, slug, file_path, status, created_at, updated_at, tags_json)
kb_entries(id, title, file_path, created_at, updated_at, tags_json, public)
kb_post_links(kb_id, post_id, link_type, created_at)  -- bidirectional
ai_conversations(id, type, input_summary, output_summary, model, created_at)
settings(key, value)
```

## Data Flow

### User edits a post
```
User types in editor
  → Milkdown updates markdown in memory
  → Auto-save to local file (Tauri fs plugin)
  → Status: "unsaved changes" indicator

User clicks "Save"
  → Write markdown file to disk
  → git add + git commit (isomorphic-git)
  → Update SQLite index
  → Status: "saved (not deployed)"

User clicks "Publish"
  → git push to GitHub (isomorphic-git)
  → Trigger Hexo generate (Node sidecar)
  → hexo deploy OR GitHub Actions triggers build
  → Status: "published ✓"
```

### AI generates post from KB
```
User selects KB entries
  → Frontend sends KB content to AI Service
  → Vercel AI SDK streams response
  → Editor populates with generated content
  → User edits/approves
  → Save flow (same as above)
  → Update kb_post_links in SQLite
```

### MCP tool call (external agent)
```
AI Agent calls MCP tool "create-post"
  → MCP Server receives request (stdio)
  → Validates input (Zod schema)
  → Creates markdown file on disk
  → git add + commit + push
  → Returns success response to agent
```

## Git Abstraction Layer

Users never see git. The abstraction provides:

| User sees | Git reality |
|-----------|-------------|
| "Save" | `git add` + `git commit` |
| "Publish" | `git push` + deploy trigger |
| "Version history" | `git log` + `git diff` |
| "Undo" | `git revert` or `git checkout -- file` |
| "Sync" | `git pull --rebase` |
| Conflict | Visual merge UI (show both versions) |

### Conflict Strategy
- **Prevention first:** Single-user app rarely has conflicts
- **Detection:** On pull, check for merge conflicts
- **Resolution UI:** Show "Your version" vs "Remote version" side-by-side
- **Fallback:** Keep both versions, let user choose

## AI/LLM Layer (Multi-Model Adapter)

```typescript
// Unified interface via Vercel AI SDK provider registry
const ai = createProviderRegistry({
  openrouter: createOpenRouter({ apiKey }),
  openai: createOpenAI({ apiKey }),
  anthropic: createAnthropic({ apiKey }),
  ollama: createOllama({ baseURL }),
})

// User selects model in settings
const model = ai.languageModel(userSettings.selectedModel)
// e.g., "openrouter:anthropic/claude-4-sonnet"

// All AI operations use the same interface
const result = await streamText({ model, prompt, system })
```

## MCP Server Integration

**Architecture:** Separate process, communicates with main app.

```
[Claude Desktop / Cursor]
    ↕ stdio
[open-blog-mcp-server]  (npm package, global install)
    ↕ local file system
[Blog repo on disk]     (same repo the desktop app manages)
```

The MCP server reads/writes the same git repo the desktop app uses. No network communication between MCP server and desktop app needed — they share the file system.

**Registration:** Users add to `claude_desktop_config.json`:
```json
{
  "mcpServers": {
    "open-blog": {
      "command": "npx",
      "args": ["@open-blog/mcp-server", "--repo", "/path/to/blog"]
    }
  }
}
```

## Authentication & GitHub Integration

### OAuth Flow (for non-technical users)
1. User clicks "Connect GitHub" in onboarding
2. App opens browser to GitHub OAuth authorize URL
3. User grants permissions (repo, user:email)
4. Callback URL → app captures auth code
5. Exchange code for access token (via app's OAuth app)
6. Store token securely (Tauri keychain/credential store)

### For MVP (simpler)
- Use GitHub Personal Access Token (PAT)
- Guide user to create PAT with `repo` scope
- Store in Tauri secure storage (tauri-plugin-store with encryption)

### Repo Creation
```
User enters blog name in wizard
  → Create repo via GitHub API (POST /user/repos)
  → Initialize with Hexo template
  → Configure GitHub Pages
  → Clone locally
  → Done
```

## Offline/Sync Strategy

| Scenario | Behavior |
|----------|----------|
| Normal (online) | Edit → save → commit → push immediately |
| Offline | Edit → save → commit locally. Push when online. |
| Reconnect | Auto-push queued commits |
| Conflict on push | Pull + merge/rebase, show conflict UI if needed |

**Key principle:** All data is local-first. GitHub is the sync/deploy target, not the source of truth. The local git repo is always authoritative.

## Suggested Build Order

| Phase | Components | Rationale |
|-------|-----------|-----------|
| 1 | Tauri scaffold + React + Vite | Foundation |
| 2 | GitHub auth + repo clone | Enables all git features |
| 3 | Markdown editor (Milkdown) + file save | Core user value |
| 4 | Post CRUD + frontmatter UI | Complete blog management |
| 5 | Git commit + push + deploy pipeline | Makes it "real" |
| 6 | Onboarding wizard (new user flow) | Non-tech user gate |
| 7 | KB system + SQLite | Enables AI features |
| 8 | AI integration (Vercel AI SDK) | KB→Post generation |
| 9 | MCP Server | External agent access |
| 10 | Desktop packaging + distribution | Ship it |

Dependencies: 1→2→3→4→5 are sequential. 6 can parallel with 4-5. 7→8→9 are sequential.
