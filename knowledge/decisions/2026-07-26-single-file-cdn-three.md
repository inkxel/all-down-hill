---
type: Decision
date: 2026-07-26
description: The whole game ships as one index.html with Three.js from a CDN — no build step, no addons, no second dependency.
status: accepted
deciders: Tucker
related: [[deploy]], [[index-map]]
---

# Decision: One self-contained HTML file, Three.js from CDN only

## Context
Requirement was a complete 3D endless snowboarder deployable as a static file on
GitHub Pages, with everything procedural and no external assets.

## Decision
A single `index.html` containing markup, CSS and one ES module `<script>`.
Three.js is imported directly from a version-pinned jsDelivr URL
(`three@0.185.1/build/three.module.js`) — a direct module import rather than an
import map, since import maps need iOS Safari 16.4+ while plain module imports
work far further back.

## Consequences
- **Positive:** deploy is `git push`. No toolchain to rot, no lockfile, no CI.
- **Positive:** the file is the artifact — what you read is exactly what runs.
- **Negative:** no three.js addons. `BufferGeometryUtils.mergeGeometries` would be a second network module, so geometry merging is hand-rolled (~18 lines).
- **Negative:** no unit-test surface. All verification is browser-driven via Playwright.
- **Neutral:** the CDN version is pinned; upgrades are a deliberate edit.

## Dissent / Alternatives Considered
- **Import map + bare `three` specifier** — cleaner-looking, but adds an iOS 16.4 floor for no functional gain.
- **Vendor three.js into the repo** — removes the CDN dependency but puts 650 KB of third-party code in the diff and still needs a copy step.
- **A real bundler (Vite)** — rejected outright; the requirement was explicitly a single self-contained file.

## Sources
- [[journal/2026-07-26-build-and-review]]
