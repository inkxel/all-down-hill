---
name: deploy
description: How the game ships — static GitHub Pages from main branch root, no build step, and what that constrains.
type: subsystem
created: 2026-07-27
last_updated: 2026-07-27
confidence: high
related: [[decisions/2026-07-26-single-file-cdn-three]]
---

# Deploy

**Live:** https://inkxel.github.io/all-down-hill/
**Repo:** `inkxel/all-down-hill` (public)

GitHub Pages, `main` branch, root path. There is no build, no CI, no bundler —
`index.html` is the artifact. Committing to `main` and pushing is the deploy.

Pages typically takes 30–90 s to serve a new commit. To confirm a deploy has
actually landed rather than trusting the build status, grep the live URL for a
string you just added:

```bash
until [ "$(curl -s https://inkxel.github.io/all-down-hill/ | grep -c <new-symbol>)" != "0" ]; do sleep 5; done
```

## What the single-file constraint costs

- No npm, so **no three.js addons** — anything under `three/addons/` would be a second network module. Geometry merging is hand-rolled for this reason.
- The Three.js CDN URL is version-pinned (`three@0.185.1`). Un-pinning would make the game hostage to upstream breaking changes with no lockfile to catch it.
- All testing is browser-driven. There is no unit-test surface; verification is Playwright against a local `python3 -m http.server` and then against the live URL.

## Context log

### 2026-07-27 — Article created
[[journal/2026-07-26-build-and-review]]
