---
title: Roadmap
type: Reference
---

# Roadmap

The "not now, but don't forget" pile.

## Awaiting Tucker's feel notes
- **Rider pose constants are provisional.** Shipped Ink to get it in his hands; crouch depth, arm swing and scarf sweep are all first-pass numbers.
- **`RIDER_SCALE = 1.15`** is a readability compromise, not a considered proportion. Real scale is 1.0.

## Known-soft
- **Section rhythm is fixed-period (350 m).** A long session will feel the loop. Density ramping masks it; it doesn't solve it.
- **Barrel roll is hard to land.** `ROLL_RATE` is 400°/s against a 40° window — a full rotation needs release timing within ~100 ms. May want to snap toward the nearest 360° on release.
- **Small kickers at ±14 units are still easy to miss** riding a straight line. The full-width feature is the only guaranteed hit.

## Deferred features
- **Rider treatments 2–5** are specified and rendered but unbuilt. Halo needs its vertex-colour rim bake restored; the rim is baked against a fixed light direction so it won't follow the day cycle without a shader.
- **Leaderboard is `localStorage` only.** No accounts, no server, no sharing beyond the JPEG card.
- **No pause.** Deliberate per the original spec, but the continue panel now proves a pause surface would fit.

## Technical debt
- **The generated `_codemap.md` is empty** because the program is one HTML file and `codemap.py` doesn't descend into inline `<script>`. [[index-map]] is the hand-maintained substitute and will drift.
- **No unit-test surface.** All verification is Playwright against a running browser. Fine at this size; would not scale.
- **Three.js is pinned to `0.185.1`.** No lockfile, so an upgrade is a manual edit and a full manual re-verify.
