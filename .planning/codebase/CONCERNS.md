# Concerns

## High Priority

### 1. node_modules tracked in git
The `node_modules/` directory appears to be tracked in the repository despite being in `.gitignore`. This bloats the repo significantly and makes git operations slower. The `.gitignore` should prevent this, but existing tracked files need `git rm --cached` to remove.

### 2. Broken site URL
`_config.yml` has `url: https://giyanwei.github.io.git` — the `.git` suffix is incorrect for a URL. Should be `https://giyanwei.github.io`.

### 3. Multiple deleted posts in working tree
Git status shows several deleted posts (`Devops.md`, `My-Journey-to-Apply-for-Master-s.md`, `hello-world.md`, `What-I-ve-learned-in-TongJi.md`, etc.) — these changes are unstaged and may be intentional or accidental.

## Medium Priority

### 4. Theme as unregistered submodule
`themes/butterfly/` contains its own `.git` directory, making it a nested git repo. It's not registered as a proper git submodule (no `.gitmodules` file), which can cause confusion during cloning.

### 5. No CI/CD pipeline
No GitHub Actions workflow for automated build/deploy. Deployment relies on manual `hexo deploy` command.

### 6. Avatar typo
In `themes/butterfly/_config.yml`, `effect: ture` should be `effect: true`.

## Low Priority

### 7. Landscape theme config leftover
`_config.landscape.yml` exists but the active theme is Butterfly. This is a default Hexo scaffold leftover.

### 8. No categories page in navigation
The Categories page exists (`source/Categories/index.md`) but is commented out in the navigation menu.

### 9. Social share dependency
`social-share.js` package is listed as a dependency but its integration status is unclear.
