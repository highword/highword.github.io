# Stack Research: Open-Blog

**Project:** Open-Blog (AI-native blog management platform)
**Researched:** 2026-06-12
**Overall confidence:** HIGH

## Recommended Stack (Summary Table)

| Layer | Technology | Version | Rationale |
|-------|-----------|---------|-----------|
| Frontend Framework | React 19 + Vite | 19.2.x / 8.x | Largest ecosystem, best Tauri integration, Milkdown/TipTap bindings |
| State Management | Zustand + TanStack Query | 5.x / 5.x | Lightweight, no boilerplate, async data caching |
| Desktop Wrapper | Tauri 2 | 2.11.x | 600KB bundle, native perf, iOS/Android support, Rust backend |
| Backend (Local) | Tauri Rust Core + Node sidecar | - | Git ops in sidecar, heavy compute in Rust |
| Git Integration | isomorphic-git (primary) + simple-git (fallback) | 1.38.x / 3.36.x | Pure JS, browser-compatible, no native deps |
| AI / LLM | Vercel AI SDK 6 | 6.x | LanguageModelV4, provider registry, streaming, tool calling |
| AI Router | OpenRouter provider | 2.9.x | Single API key, 200+ models, fallback routing |
| MCP Server | @modelcontextprotocol/server | 2.0.x | Official TS SDK, Streamable HTTP + stdio transport |
| Markdown Editor | Milkdown (Crepe) | 7.21.x | Typora-like WYSIWYG, plugin system, React bindings, AI feature |
| Database | Drizzle ORM + better-sqlite3 | 0.45.x / 12.x | Type-safe, zero-dep at runtime, local-first |
| Styling | Tailwind CSS 4 | 4.x | Utility-first, Tauri-friendly, no runtime cost |

---

## Frontend Framework

### Recommendation: React 19 + Vite 8

**Confidence: HIGH**

**Why React over Vue/Svelte:**

1. **Tauri official support:** Tauri's `create-tauri-app` provides first-class React templates. The Tauri docs specifically recommend "Vite for SPA frameworks such as React, Vue, Svelte" — all are equal here, but React wins on ecosystem.

2. **Editor component availability:** Both Milkdown and TipTap provide official React bindings (`@milkdown/react`, `@tiptap/react`). Svelte bindings are community-maintained and less stable.

3. **AI SDK integration:** Vercel AI SDK provides `useChat`, `useCompletion`, and `useObject` hooks designed for React. Vue/Svelte support exists but is second-class.

4. **Cross-platform path:** React Native (via Tauri mobile) shares mental models. If mobile-native is ever needed beyond Tauri's webview, React Native is an option.

5. **Ecosystem depth:** MCP tooling examples, GitHub integrations, and community patterns all default to React.

**Why NOT Next.js:** Next.js is a server-side framework. Open-Blog is a local-first desktop app — no Node.js server at runtime. Vite provides the SPA bundling without SSR overhead. Tauri docs explicitly say "meta-frameworks require special configuration."

**Why Vite 8 over Webpack:** Faster HMR, simpler config, Tauri's recommended bundler.

### Key Packages

```
react@19.2.7
react-dom@19.2.7
vite@8.0.16
@vitejs/plugin-react@4.x
typescript@5.x
```

### State Management

| Library | Purpose |
|---------|---------|
| `zustand@5.x` | UI state, settings, editor state |
| `@tanstack/react-query@5.x` | Async data: git operations, AI responses, file system |

**Why Zustand over Redux/Jotai:** Minimal API, no providers needed, works perfectly in Tauri webview context. Excellent TypeScript support. No boilerplate.

---

## Backend / Local Server

### Recommendation: Tauri 2 Rust Core + Node.js Sidecar (for Hexo)

**Confidence: HIGH**

**Architecture:**

```
[React Frontend] <-- IPC (Commands + Events) --> [Tauri Rust Core]
                                                       |
                                                       +--> [Shell Plugin: Node sidecar for Hexo CLI]
                                                       +--> [Rust: File system, SQLite, git bindings]
                                                       +--> [HTTP: AI API calls via frontend fetch]
```

**Why this hybrid approach:**

1. **Tauri 2 IPC is fast:** Uses custom protocols (not string serialization like v1). Commands are async Rust functions callable from JS.

2. **Hexo requires Node.js:** Hexo is a Node.js static site generator. The Tauri Shell plugin supports sidecars — bundled executables that run alongside the app. Bundle a Node.js runtime + Hexo CLI as a sidecar.

3. **Git operations run in the frontend:** isomorphic-git runs in the webview (pure JS), no backend needed for basic git ops. For heavy operations (large repos), fall back to a Rust git2 binding via Tauri command.

4. **AI calls from frontend:** HTTP requests to LLM APIs go directly from the webview via fetch (Tauri allows it with CSP config). No backend proxy needed.

**Why NOT a separate Express/Fastify server:** Adds complexity, another process to manage, port conflicts. Tauri's IPC handles everything a local server would do, natively.

**Why NOT pure Rust backend for everything:** Hexo is Node.js-only. The team likely has more TypeScript expertise than Rust. Keep Rust thin (file system, SQLite, performance-critical paths).

### Tauri Plugin Configuration

```json
{
  "plugins": {
    "shell": { "open": true },
    "fs": { "scope": ["$APPDATA/**", "$HOME/**"] },
    "dialog": {},
    "notification": {}
  }
}
```

---

## Git Integration Library

### Recommendation: isomorphic-git (primary) + simple-git (sidecar fallback)

**Confidence: HIGH**

### isomorphic-git (Primary)

| Aspect | Detail |
|--------|--------|
| Version | 1.38.4 |
| Runs in | Browser (webview) AND Node.js |
| Dependencies | Zero native deps |
| Operations | clone, commit, push, pull, log, status, add, branch, checkout |
| Auth | onAuth callback (GitHub token) |
| Limitations | No rebase, no submodules, no LFS |

**Why isomorphic-git:**

1. **Runs in Tauri webview** — no IPC overhead for git operations
2. **Pure JavaScript** — no native compilation issues across platforms
3. **GitHub token auth** — `onAuth: () => ({ username: token })` pattern
4. **Progress tracking** — `onProgress` callback for clone/push UIs
5. **Browser filesystem** — works with LightningFS (for web-only mode) or Node fs (via sidecar)

**Code pattern for Tauri:**
```typescript
import git from 'isomorphic-git'
import http from 'isomorphic-git/http/web'

// In Tauri, use @tauri-apps/plugin-fs for filesystem access
// isomorphic-git accepts any fs-compatible interface
await git.clone({
  fs,  // Tauri fs adapter
  http,
  dir: '/path/to/repo',
  url: 'https://github.com/user/blog.git',
  onAuth: () => ({ username: githubToken }),
  onProgress: (e) => updateProgressBar(e),
  singleBranch: true,
  depth: 10
})
```

### simple-git (Fallback via Sidecar)

| Aspect | Detail |
|--------|--------|
| Version | 3.36.0 |
| Runs in | Node.js only |
| Use case | Complex operations isomorphic-git can't handle (rebase, stash, LFS) |

**When to use simple-git:** If the user's repo requires `git lfs`, `git rebase`, or other advanced operations, spawn a system git process via Tauri's shell plugin. simple-git wraps the CLI git binary.

### Why NOT nodegit

- Native C++ bindings = compilation hell on Windows
- Abandoned/unmaintained (last meaningful update years ago)
- Bundle size explosion
- Incompatible with Tauri's webview model

---

## AI / LLM Integration

### Recommendation: Vercel AI SDK 6 + OpenRouter Provider

**Confidence: HIGH**

### Vercel AI SDK

| Aspect | Detail |
|--------|--------|
| Version | 6.0.202 (latest) |
| Architecture | LanguageModelV4 interface + provider pattern |
| Key APIs | `generateText`, `streamText`, `generateObject`, `streamObject` |
| Streaming | First-class, async iterables |
| Tool Calling | Built-in with Zod schema validation |
| Framework | Framework-agnostic core (works without Next.js) |

**Why Vercel AI SDK over LangChain.js:**

1. **Lighter weight:** AI SDK is ~50KB, LangChain is 500KB+ with all its abstractions
2. **Provider abstraction:** LanguageModelV4 interface — swap providers without code changes
3. **Streaming native:** Built for streaming from day one, not bolted on
4. **Type safety:** Full TypeScript with Zod schema for structured output
5. **No opinion on orchestration:** AI SDK provides the primitives; you build the orchestration. LangChain enforces chains/agents patterns you may not need.
6. **Active development:** v6 is current, rapid releases, Vercel-backed

**Why NOT LangChain.js:**
- Over-abstraction for this use case (we're calling LLMs, not building RAG pipelines)
- Heavy bundle size
- Frequent breaking changes between versions
- "Framework tax" — forces patterns that don't fit local-first apps

### Provider Strategy

```typescript
import { createProviderRegistry, gateway } from 'ai'
import { createOpenRouter } from '@openrouter/ai-sdk-provider'

// Single provider for all models via OpenRouter
const openrouter = createOpenRouter({
  apiKey: userSettings.openrouterKey,
})

// OR direct providers for power users
import { openai } from '@ai-sdk/openai'
import { anthropic } from '@ai-sdk/anthropic'
import { google } from '@ai-sdk/google'

const registry = createProviderRegistry({
  openrouter,
  openai,
  anthropic,
  google,
})
```

### OpenRouter as Default Router

| Aspect | Detail |
|--------|--------|
| Version | @openrouter/ai-sdk-provider@2.9.1 |
| Models | 200+ models from all major providers |
| Benefit | User needs ONE API key for all models |
| Fallback | Built-in model fallback routing |
| Cost | Pay-per-token, no subscription |

**For non-technical users:** OpenRouter is the right default. One signup, one API key, access to GPT-4o, Claude, Gemini, Llama, etc. Power users can add direct provider keys.

### AI SDK Packages

```
ai@6.0.202
@ai-sdk/openai@3.0.70
@ai-sdk/anthropic@3.0.83
@ai-sdk/google@3.0.81
@openrouter/ai-sdk-provider@2.9.1
```

---

## MCP Server SDK

### Recommendation: @modelcontextprotocol/server (TypeScript SDK)

**Confidence: HIGH**

| Aspect | Detail |
|--------|--------|
| Package | @modelcontextprotocol/server |
| Version | 2.0.0-alpha.2 (latest) |
| Transports | Stdio, Streamable HTTP, SSE (legacy) |
| Features | Tools, Resources, Prompts |
| Schema | Zod v4 for input validation |

**Architecture for Open-Blog MCP:**

Open-Blog should expose an MCP server so AI agents (Claude, etc.) can:
1. **Read blog posts** (resource: list posts, get post content)
2. **Create/edit posts** (tool: create-post, update-post)
3. **Manage KB** (tool: search-kb, add-to-kb)
4. **Deploy** (tool: deploy-to-github-pages)

```typescript
import { McpServer } from '@modelcontextprotocol/server'
import { StdioServerTransport } from '@modelcontextprotocol/server/stdio'
import * as z from 'zod/v4'

const server = new McpServer({ 
  name: 'open-blog', 
  version: '1.0.0' 
})

server.registerTool('create-post', {
  description: 'Create a new blog post',
  inputSchema: z.object({
    title: z.string(),
    content: z.string(),
    tags: z.array(z.string()).optional(),
  }),
}, async ({ title, content, tags }) => {
  // Create hexo post file, git commit
  return { content: [{ type: 'text', text: `Created: ${title}` }] }
})

// Stdio transport for local MCP client integration
const transport = new StdioServerTransport()
await server.connect(transport)
```

**Transport choice:**
- **Stdio** for local integration (Claude Desktop, Cursor, etc.)
- **Streamable HTTP** for future remote/web access

### MCP Packages

```
@modelcontextprotocol/server@2.0.0-alpha.2
zod@^3.25 (zod/v4 subpath)
```

---

## Desktop Wrapper

### Recommendation: Tauri 2

**Confidence: HIGH**

| Aspect | Tauri 2 | Electron |
|--------|---------|----------|
| Bundle size | ~600KB - 3MB | ~80-150MB |
| RAM usage | 30-80MB | 150-500MB |
| Startup time | <1s | 2-5s |
| Mobile support | iOS + Android (stable) | None |
| Backend language | Rust | JavaScript (Node.js) |
| Webview | System (WebView2/WebKit) | Bundled Chromium |
| Security | Capability-based permissions | Open by default |
| Version | 2.11.2 | 36.x |

**Why Tauri 2 over Electron:**

1. **Mobile support:** Tauri 2 ships iOS and Android builds from the same codebase. The requirement states "eventually mobile" — Tauri delivers this.

2. **Bundle size:** Blog management app should be lightweight. 600KB vs 150MB is night and day for distribution.

3. **Performance:** System webview means lower RAM. Users run this alongside VS Code, Chrome, etc.

4. **Security model:** Capability-based permissions prevent frontend from accessing anything not explicitly granted. Critical for an app handling git credentials.

5. **Shell plugin:** Native support for spawning sidecars (Hexo CLI), exactly what we need.

6. **Modern IPC:** v2's custom protocol IPC is fast enough for real-time editor synchronization.

**Tauri 2 platform targets:**
- Windows (WebView2 - auto-installs)
- macOS (WebKit - system-included)
- Linux (WebKitGTK - package dependency)
- iOS (WKWebView - system-included)
- Android (Android WebView - system-included)

**Potential concern:** System webview means slight CSS/rendering differences across platforms. Mitigate with Tailwind (utility classes normalize well) and testing on all platforms.

### Tauri Packages

```
@tauri-apps/cli@2.11.2
@tauri-apps/api@2.11.0
@tauri-apps/plugin-shell@2.x
@tauri-apps/plugin-fs@2.x
@tauri-apps/plugin-dialog@2.x
@tauri-apps/plugin-notification@2.x
@tauri-apps/plugin-store@2.x
```

---

## Markdown Editor

### Recommendation: Milkdown (Crepe)

**Confidence: HIGH**

| Aspect | Milkdown (Crepe) | TipTap | Monaco |
|--------|-----------------|--------|--------|
| WYSIWYG Markdown | Native (Typora-like) | Plugin (add-on) | No (code editor) |
| React bindings | Official `@milkdown/react` | Official `@tiptap/react` | Via wrapper |
| Bundle size | ~150KB | ~200KB | ~2MB |
| Plugin system | First-class | First-class | Limited |
| AI integration | Built-in feature flag | Pro (paid) | None |
| Slash commands | Built-in (Crepe) | Extension | None |
| Toolbar | Built-in (Crepe) | DIY | Built-in |
| License | MIT | MIT (core) / Paid (pro) | MIT |
| Markdown fidelity | Lossless (remark-based) | Lossy (ProseMirror) | Perfect (raw) |

**Why Milkdown over TipTap:**

1. **Markdown-first:** Milkdown is built on remark (unified ecosystem). Markdown is the source of truth, not an export format. For a Hexo blog manager, markdown fidelity is critical.

2. **Crepe = batteries included:** Toolbar, slash commands, block editing, image blocks, code mirror, LaTeX, tables — all built-in with `@milkdown/crepe`. No need to assemble 20 extensions.

3. **AI feature built-in:** Crepe has a `Crepe.Feature.AI` flag (disabled by default). This provides an integration point for AI-powered writing assistance.

4. **Free & MIT:** TipTap's best features (collaboration, AI, comments) are paid. Milkdown is fully open source.

5. **Typora-like experience:** The target audience (non-technical users) expects Typora/Notion-like editing. Milkdown delivers this out of the box.

**Why NOT TipTap:**
- Markdown support is an afterthought (HTML-first)
- AI features require TipTap Pro subscription
- Collaboration features are paid
- For a markdown-native app, TipTap's HTML-first model creates impedance mismatch

**Why NOT Monaco:**
- Code editor, not rich text editor
- Non-technical users won't understand raw markdown editing
- 2MB bundle for something that doesn't match the UX goal

### Milkdown Packages

```
@milkdown/crepe@7.21.2
@milkdown/react@7.x
@milkdown/kit@7.x
@milkdown/theme-nord@7.x (or custom theme)
```

### Editor Architecture

```
[Milkdown Crepe Editor]
    |-- Toolbar (bold, italic, link, code, AI assist)
    |-- Slash Menu (headings, lists, code blocks, images, tables)
    |-- Block Edit (drag handles, plus button)
    |-- CodeMirror (syntax highlighting in code blocks)
    |-- Image Block (upload/paste with preview)
    |-- Frontmatter Plugin (custom: Hexo YAML frontmatter)
```

---

## Database / Storage

### Recommendation: Drizzle ORM + better-sqlite3

**Confidence: HIGH**

| Aspect | Detail |
|--------|--------|
| ORM | Drizzle ORM 0.45.2 |
| Driver | better-sqlite3 12.10.0 |
| Location | `$APPDATA/open-blog/data.db` |
| Purpose | KB metadata, post index, settings, AI conversation history |

**Why SQLite:**
- Local-first app — no database server
- Single file, easy backup/sync
- ACID compliant
- Handles millions of records for KB
- Works in Tauri via Rust (rusqlite) or Node sidecar

**Why Drizzle over Prisma:**
- Lighter weight (no binary engine like Prisma)
- SQL-like API (not abstracted away)
- Better SQLite support
- Faster migrations
- Works in sidecar Node.js process

**Schema example:**
```typescript
import { sqliteTable, text, integer } from 'drizzle-orm/sqlite-core'

export const posts = sqliteTable('posts', {
  id: text('id').primaryKey(),
  title: text('title').notNull(),
  slug: text('slug').notNull().unique(),
  filePath: text('file_path').notNull(),
  status: text('status', { enum: ['draft', 'published'] }).notNull(),
  createdAt: integer('created_at', { mode: 'timestamp' }).notNull(),
  updatedAt: integer('updated_at', { mode: 'timestamp' }).notNull(),
  tags: text('tags'), // JSON array
})

export const kbEntries = sqliteTable('kb_entries', {
  id: text('id').primaryKey(),
  title: text('title').notNull(),
  content: text('content').notNull(),
  embedding: text('embedding'), // JSON float array for vector search
  sourceUrl: text('source_url'),
  createdAt: integer('created_at', { mode: 'timestamp' }).notNull(),
})

export const settings = sqliteTable('settings', {
  key: text('key').primaryKey(),
  value: text('value').notNull(),
})
```

**Alternative for Tauri-native:** Consider `tauri-plugin-sql` which wraps rusqlite and exposes SQL operations via IPC. This eliminates the need for a Node sidecar just for DB operations.

### Database Packages

```
drizzle-orm@0.45.2
better-sqlite3@12.10.0
drizzle-kit@0.x (dev, for migrations)
@types/better-sqlite3@7.x (dev)
```

---

## What NOT to Use (Anti-Recommendations)

| Technology | Why NOT |
|------------|---------|
| **Next.js** | Server framework; Open-Blog is a local desktop app. No SSR needed. Adds massive complexity for zero benefit in Tauri. |
| **Electron** | 150MB bundle, no mobile support, higher RAM. Tauri 2 is strictly superior for this use case. |
| **Prisma** | Requires binary engine (~15MB), heavyweight for local SQLite. Drizzle is lighter. |
| **LangChain.js** | Over-engineered for "call LLM and stream response." 500KB+ bundle. Use AI SDK instead. |
| **nodegit** | Native C++ deps, compilation hell, abandoned. Use isomorphic-git. |
| **Monaco Editor** | Code editor, not rich text. Wrong UX for non-technical users. |
| **Redux / MobX** | Over-engineering for a local app. Zustand does the same in 1/10 the code. |
| **Express/Fastify** | No separate HTTP server needed. Tauri IPC handles backend communication. |
| **Webpack** | Slow, complex config. Vite is faster and simpler for Tauri apps. |
| **TipTap Pro** | Paid features (AI, collab) that Milkdown provides for free. |
| **Firebase/Supabase** | Cloud-first services. Open-Blog is local-first, git-native. No cloud DB needed. |
| **Capacitor/Ionic** | Mobile frameworks that add abstraction layers. Tauri 2 handles mobile natively. |

---

## Additional Supporting Libraries

| Library | Version | Purpose |
|---------|---------|---------|
| `zod` | 3.25.x | Schema validation (AI SDK, MCP, forms) |
| `date-fns` | 4.x | Date formatting for blog posts |
| `gray-matter` | 4.x | Parse Hexo frontmatter (YAML) |
| `@tauri-apps/plugin-store` | 2.x | Persistent key-value settings |
| `lucide-react` | latest | Icon set |
| `tailwindcss` | 4.x | Styling |
| `react-hot-toast` | latest | Notifications |
| `@tanstack/react-router` | 1.x | Type-safe routing (or TanStack Router) |

---

## Installation Commands

```bash
# Create Tauri app with React + Vite
npm create tauri-app@latest open-blog -- --template react-ts

# Frontend dependencies
npm install react@19 react-dom@19 zustand@5 @tanstack/react-query@5
npm install @milkdown/crepe @milkdown/react @milkdown/kit
npm install ai @ai-sdk/openai @ai-sdk/anthropic @openrouter/ai-sdk-provider
npm install @modelcontextprotocol/server zod
npm install isomorphic-git
npm install drizzle-orm better-sqlite3
npm install gray-matter date-fns lucide-react
npm install @tanstack/react-router

# Tauri plugins
npm install @tauri-apps/plugin-shell @tauri-apps/plugin-fs
npm install @tauri-apps/plugin-dialog @tauri-apps/plugin-notification
npm install @tauri-apps/plugin-store

# Dev dependencies
npm install -D typescript @types/react @types/react-dom
npm install -D tailwindcss @tailwindcss/vite
npm install -D drizzle-kit @types/better-sqlite3
npm install -D vite @vitejs/plugin-react
npm install -D @tauri-apps/cli
```

---

## Sources

- Tauri 2 official docs: https://v2.tauri.app/ (Context7 verified)
- Vercel AI SDK: https://github.com/vercel/ai (Context7 verified, LanguageModelV4 architecture)
- MCP TypeScript SDK: https://github.com/modelcontextprotocol/typescript-sdk (Context7 verified, v1.29.0+)
- isomorphic-git: https://github.com/isomorphic-git/isomorphic-git (Context7 verified)
- Milkdown: https://milkdown.dev/ (Context7 verified, Crepe API)
- Drizzle ORM: https://orm.drizzle.team/ (Context7 verified, SQLite support)
- OpenRouter AI SDK Provider: https://github.com/openrouterteam/ai-sdk-provider (Context7 verified)
- npm registry: package versions verified via `npm view` on 2026-06-12
