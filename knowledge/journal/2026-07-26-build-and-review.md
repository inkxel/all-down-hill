---
type: Journal
date: 2026-07-26
session: build-and-review
description: Built the game end to end in one file, deployed to Pages, then ran a correctness pass (Codex) and a complexity pass (ponytail-review).
status: shipped
related: [[terrain]], [[physics]], [[atmosphere]], [[deploy]]
---

# Build and review — first session

## Context
Greenfield. Detailed spec from Tucker: 3D endless snowboarder, Alto's Adventure
as reference but third-person chase camera, single self-contained HTML file,
Three.js from CDN, everything procedural. Ponytail was explicitly turned **off**
for the build phase — the spec deliberately asks for effects a YAGNI ladder would
strip — and reserved for the review phase.

## What changed
- Created `inkxel/all-down-hill`, shipped `index.html` (~2,000 lines), README, GitHub Pages from main root.
- Subsystems: [[terrain]], [[physics]], [[atmosphere]], [[rider]], [[deploy]].
- Live: https://inkxel.github.io/all-down-hill/

## Decisions made
- [[decisions/2026-07-26-single-file-cdn-three]]
- [[decisions/2026-07-26-analytic-terrain-height]]
- [[decisions/2026-07-26-fixed-slot-instancing]]
- [[decisions/2026-07-26-light-intensity-pi-scale]]
- [[decisions/2026-07-26-float32-rebasing]]

## Bugs found by looking, not by review
Three things the automated passes never would have caught, all surfaced by
screenshotting the running game:

1. **Everything ~3× too dark.** three.js r155 dropped legacy π light scaling. Diagnosed by computing the expected Lambert output and comparing to sampled pixels rather than guessing.
2. **Sun swept behind the fall line.** Correct lighting maths, wrong art direction — `dot(N, L)` went negative for much of the cycle so the snow was ambient-only. Narrowed the azimuth arc.
3. **Rocks fell apart.** `IcosahedronGeometry` is already non-indexed, so jittering per-vertex split every shared corner into loose triangles. Fixed with a position-hash so duplicate corners jitter identically.

## Review pass 1 — Codex (correctness)
Scoped strictly to runtime errors, physics/math bugs, per-frame allocation,
chunk-recycling leaks, mobile input edge cases and Pages-specific breakage.
Explicitly told not to propose refactors, file splitting or tooling.

Applied 6 of 8 — see [[research/2026-07-review-passes]] for the full ledger
including the two declined and why.

Highest-value find: **obstacle collision was a point test**, which tunnels at
42 u/s when dt hits the 50 ms clamp (2.1 units of travel vs a ~1.2 unit tree
radius). Replaced with a swept segment test.

## Review pass 2 — ponytail-review (complexity)
Treated as advisory. Applied the genuine redundancy (hand-rolled `clamp`/`lerp`/
`smoothstep`/`damp` that `THREE.MathUtils` already ships, a duplicated re-base
block, values derived in multiple per-frame functions). Declined two on
dependency grounds, not sentiment. Net −16 lines.

**This pass introduced a latent bug.** Swapping to `MathUtils.smoothstep`
changed the argument convention from `(edge0, edge1, x)` to `(x, min, max)` —
which mattered the next session when an inverted smoothstep silently returned 0.
See [[physics]].

## Open threads
- [ ] Tucker to play and give feel notes.

## Related
- [[research/2026-07-review-passes]]
