---
name: terrain
description: Infinite slope from layered simplex noise, queried analytically and rendered from a recycled pool of nine chunks with a paced prop rhythm.
type: subsystem
created: 2026-07-27
last_updated: 2026-07-27
confidence: high
related: [[physics]], [[index-map]], [[decisions/2026-07-26-analytic-terrain-height]], [[decisions/2026-07-27-section-based-pacing]]
---

# Terrain — generation, chunking, pacing

## The height function

`terrainHeight(x, z)` is a pure function of world position. There is no stored
heightfield of record; the mesh is a *rendering* of the function, and physics
queries the function directly. See
[[decisions/2026-07-26-analytic-terrain-height]].

Composition:
- A base fall line, `z * SLOPE` where `SLOPE = 0.36` (≈19.8°). Travel is toward **−z**, so decreasing z lowers y.
- Three octaves of simplex noise (amplitudes 8.0 / 3.0 / 1.0).
- A parabolic bowl in x plus a linear berm beyond |x| = 34, which is what keeps you on the mountain visually.

The amplitude/frequency pairs were chosen so the worst-case combined gradient
stays under `tan(40°)`, satisfying the "nothing steeper than 40 degrees"
constraint. `INFERRED` — the bound is analytic, not measured.

`groundAt(x, z) = terrainHeight(x, z) + rampRise(x, z)`. **Physics must always use
`groundAt`; visuals that should ignore kickers (the carve trail) use
`terrainHeight`.**

## Chunks

Nine `PlaneGeometry` chunks, 140 × 50 units at 2-unit resolution, in a fixed pool.
`updateChunks` finds the rearmost chunk and rebuilds it ahead of the player —
positions, vertex colours, bounding sphere, and props are all rewritten in place.
Nothing is allocated or disposed after startup. Verified: heap is flat at 9.5 MB
across a 30-second soak.

Vertex colours carry the groomed-corduroy striping (2-metre bands keyed on
`Math.floor(x / 2) & 1`) and a low-frequency noise grain. The day/night tint is
*not* baked in — it comes from `snowMat.color`, so the palette can change without
touching a single vertex.

## Prop pacing

Props are not scattered uniformly. `SECTIONS` is a seven-entry cycle (350 m) that
reads: slalom → tighter slalom → run-up → small kickers → run-up → one full-width
feature → landing. Density scales on top of the rhythm with distance travelled.
See [[decisions/2026-07-27-section-based-pacing]].

Ramps are placed first; trees and rocks reject any position within a pad of a
ramp footprint, so a kicker always has a clean approach and landing.

## Gotcha: ramp footprints are rotated

Angled kickers carry a yaw. `rampRise` rotates the query point into ramp space
before testing the footprint, and tapers the rise toward the lateral edges. The
taper is not cosmetic — see [[physics]] for the catapult it prevents.

## Context log

### 2026-07-27 — Article created
Covers the state after the pacing rework. [[journal/2026-07-27-feel-rework]]
