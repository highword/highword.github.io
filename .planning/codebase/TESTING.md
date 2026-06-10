# Testing

## Current State
No automated testing infrastructure exists in this project.

## Build Verification
- `hexo generate` — Validates Markdown parsing and template rendering
- `hexo server` — Manual visual inspection of generated site
- No CI pipeline for build verification

## Content Validation
- No spell checking
- No link checking (broken links)
- No front matter validation
- No image reference validation

## Dependency Management
- Dependabot provides automated security/version updates via PRs
- No lock file (`package-lock.json`) visible in tracked files

## Recommendations (if testing were needed)
- `hexo generate` as CI step to catch build-breaking posts
- `markdown-link-check` for broken link detection
- Front matter linting for required fields
