# Pitfalls Research

**Domain:** AI-native, git-native blog management platform
**Researched:** 2026-06-12

## Git-Based CMS Failures

### Forestry.io (Shutdown 2023)
- **What happened:** Acquired by TinaCMS, service shut down
- **Lesson:** SaaS-dependent git CMS is fragile. Users lost their workflow overnight.
- **Prevention:** Open-source, local-first. No server dependency for core functionality.

### Netlify CMS → Decap CMS (Maintenance mode)
- **What happened:** Netlify dropped active development, community forked as Decap
- **Lesson:** Corporate-backed open source can be abandoned
- **Prevention:** Build community early, keep architecture simple enough for contributors

### Sveltia CMS (Emerging)
- **What happened:** Community rewrite of Decap with modern tech
- **Lesson:** Users want modern UX on top of git-based content
- **Prevention:** This validates our approach — the market wants what we're building

### TinaCMS (Struggling with complexity)
- **What happened:** Complex setup, steep learning curve, developer-focused
- **Lesson:** Git CMS tools that require config files and CLI fail with non-technical users
- **Prevention:** Zero-config onboarding, no CLI exposure ever

## Hiding Git From Users

| Pitfall | Warning Sign | Prevention | Phase |
|---------|-------------|------------|-------|
| Merge conflicts surfacing as cryptic errors | Push fails silently | Visual conflict resolution UI | Phase 5 |
| Force-push data loss | User has "old" version after sync | Never force-push; always pull-rebase first | Phase 2 |
| Large file bloat (.git grows huge) | Repo clone becomes slow | Git LFS for images, shallow clone | Phase 2 |
| Detached HEAD state | Edits "disappear" | Always ensure clean branch state | Phase 2 |
| Auth token expiry | Sudden "access denied" errors | Token refresh flow, clear error messaging | Phase 2 |
| Uncommitted changes on crash | Data appears lost | Auto-save to git stash on interval | Phase 3 |

### Key Insight
The #1 rule: **Never show git errors to users.** Catch every git operation error and translate to plain language:
- "push rejected" → "Someone else made changes. Let me sync first."
- "authentication failed" → "Your GitHub connection expired. Let's reconnect."
- "merge conflict" → "You and another device edited the same paragraph. Which version do you want?"

## No-Code Builder Mistakes

| Pitfall | Examples | Prevention |
|---------|----------|------------|
| Too many choices upfront | WordPress theme/plugin selection paralysis | Opinionated defaults, expand later |
| Jargon leakage | "Configure your YAML frontmatter" | Never use technical terms in UI |
| Broken preview | "Deploy to see your changes" | Real-time local preview |
| Account creation friction | Requiring 3+ accounts (GitHub + service + API) | Single GitHub OAuth, progressive API key setup |
| Error without recovery | "Something went wrong" with no fix | Every error has a "Fix this" button |
| Feature overload on day 1 | Showing all features to new users | Progressive disclosure, feature unlock |

## AI Content Generation Risks

| Risk | Impact | Mitigation | Phase |
|------|--------|-----------|-------|
| Hallucination in generated content | Factually wrong blog posts | Always show generated content as draft, require human review | Phase 8 |
| Cost explosion | Users accidentally burn API credits | Token budget limits, cost estimation before generation | Phase 8 |
| Inconsistent tone/style | Blog feels AI-written | User-defined style guide, few-shot examples from their KB | Phase 8 |
| Prompt injection via KB | Malicious KB content corrupts generation | Sanitize KB input, separate system/user prompts | Phase 8 |
| Model API downtime | Generation fails silently | Graceful fallback, offline indicator, retry with backoff | Phase 8 |
| Over-reliance on AI | Users stop thinking about content | Position AI as "draft generator," not "publisher" | UX design |

### Key Insight
AI should **never publish directly**. Flow must be: Generate → Review → Edit → Publish. The human is always in the loop.

## Local-First Architecture Risks

| Risk | Impact | Prevention | Phase |
|------|--------|-----------|-------|
| Data loss on uninstall | User loses all content | Content is in git repo (persists), warn on uninstall | Phase 1 |
| SQLite corruption | KB index lost | SQLite is only a cache; rebuild from markdown files | Phase 7 |
| Schema migration breaks | App won't start after update | Versioned migrations, fallback to re-index | Phase 7 |
| Disk space growth | Large repos consume disk | Image optimization, git gc on schedule | Phase 5 |
| Cross-device sync issues | Different states on laptop vs desktop | Git is the sync mechanism; clear "sync" UI | Phase 5 |

### Key Insight
**Never put irreplaceable data in SQLite.** SQLite is a performance cache. Source of truth is always the markdown files in git. If SQLite corrupts, re-index from files.

## Onboarding Killers

| Killer | Why Fatal | Fix |
|--------|-----------|-----|
| Requiring GitHub account creation as step 1 | Users bounce at external site | In-app guided GitHub signup with screenshots |
| Showing empty state after setup | "Now what?" feeling | Auto-create first post template, suggest topics |
| API key setup before value is shown | Users don't understand why | Free tier with basic features, API key unlocks AI |
| Multi-step setup without progress | Feels endless | Progress bar: "Step 2 of 4" with estimated time |
| Failure at any setup step | Complete abandonment | Each step must be retryable, skippable where possible |
| No immediate visual reward | No dopamine hit | Show live blog preview within 2 minutes |

### Ideal Onboarding Flow (5 minutes)
1. "Connect GitHub" (OAuth, 30 seconds)
2. "Name your blog" (text input, 10 seconds)
3. "Choose a look" (theme picker with previews, 30 seconds)
4. "Write something!" (pre-filled template, 2 minutes)
5. "Your blog is live!" (show deployed URL) ← DOPAMINE HIT

## Cross-Platform Pitfalls (Tauri 2)

| Pitfall | Platform | Mitigation |
|---------|----------|-----------|
| WebView rendering differences | Linux (WebKitGTK older) | Test on all platforms, use standard CSS |
| Missing system WebView2 | Windows (old installs) | Tauri auto-installs WebView2 via installer |
| File path differences | Windows backslashes | Use `path` APIs, never hardcode separators |
| Keychain access differences | macOS vs Windows vs Linux | Use Tauri's secure store plugin (abstracts) |
| Node.js sidecar bundling | All platforms | Use Tauri's sidecar binaries feature |
| Auto-updater differences | Each OS has different mechanism | Use Tauri's updater plugin |
| Code signing requirements | macOS (notarization), Windows (cert) | Budget for certificates, automate in CI |

### Key Insight
Start with **web-only mode** (no desktop features needed). Add Tauri desktop wrapper after core features work in browser. This de-risks platform issues.

## GitHub Dependency Risks

| Risk | Impact | Mitigation |
|------|--------|-----------|
| API rate limiting (5000/hr authenticated) | Operations slow/fail | Cache aggressively, batch operations |
| Pages build delay (1-10 min) | User thinks publish failed | Show "deploying..." status, poll build API |
| OAuth app review required (>100 users) | Blocks growth | Apply for review early in development |
| GitHub outage | App appears broken | Clear offline indicators, local operations still work |
| Token revocation | Sudden auth failure | Graceful re-auth flow, clear messaging |
| Repository size limits (5GB) | Blog can't grow | Image optimization, external image hosting option |
| API changes / deprecation | Features break | Pin API versions, monitor changelog |

### Key Insight
**GitHub is a deployment target, not a runtime dependency.** All editing/writing must work offline. GitHub is only needed for push/deploy.

## Prevention Strategies (Summary)

| Pitfall | Warning Sign | Prevention | Phase |
|---------|-------------|------------|-------|
| Git complexity leaks to users | Error messages contain git terminology | Error translation layer, never expose raw git | 2 |
| Onboarding abandonment | >50% drop-off at any step | 5-step wizard, each step <1 min | 6 |
| AI generates bad content | Users publish without reading | Mandatory review step, no auto-publish | 8 |
| Data perceived as lost | User can't find their post | Git is backup, SQLite rebuilds from files | 3 |
| Token/auth issues | "Access denied" after time passes | Token refresh, clear re-auth UX | 2 |
| Cross-platform bugs | Works on Mac, breaks on Windows | CI testing on all platforms, progressive enhancement | 10 |
| GitHub rate limits | Batch operations fail midway | Throttle, queue, cache API responses | 5 |
| Cost surprise from AI | User gets unexpected bill | Show cost estimate before generation, set limits | 8 |
| Offline confusion | User doesn't know they're offline | Clear network status indicator, queue operations | 5 |
| Large repo clone time | New user waits 5+ minutes | Shallow clone (depth=1), lazy-load history | 2 |
