---
name: index-map
description: Structural map of index.html — what lives at which section, since the whole program is one file and the tree-sitter codemap cannot parse it.
type: subsystem
created: 2026-07-27
last_updated: 2026-07-27
confidence: high
related: [[terrain]], [[physics]], [[rider]], [[atmosphere]], [[deploy]]
---

# Index map — navigating the single file

`index.html` is the entire program: markup, CSS, and ~2,000 lines of ES module
JavaScript in one `<script type="module">`. There is no `src/`, no build step,
no module graph.

**The auto-generated `_codemap.md` is empty for this repo** and that is expected —
`codemap.py` extracts symbols from source files by extension and does not descend
into inline `<script>` blocks in HTML. This article is the hand-maintained
substitute. `EXTRACTED` from banner comments in the file.

## Sections, in file order

| Section | Purpose |
|---|---|
| `utility` | `clamp`/`lerp`/`smoothstep`/`damp` destructured from `THREE.MathUtils`; `mulberry32` seeded PRNG |
| `2D simplex noise` | Inlined Gustavson simplex, seeded permutation tables |
| `tuning` | Every physics and world constant. **Start here for feel changes.** |
| `device / DOM` | Touch detection, cached element refs, persisted best score + run list |
| `TITLE SCREEN` | 2D canvas parallax mountains, dawn palette, snowfall |
| `RENDERER / SCENE` | Renderer, camera, the three lights |
| `sky dome` / `stars` / `sun-moon disc` / `horizon silhouettes` | Backdrop layers, all camera-following |
| `TERRAIN` | `terrainHeight`, `rampRise`, `groundAt`, `terrainNormal` |
| `prop geometries` | Procedural tree/rock/ramp meshes, merged and instanced |
| `chunks` | Chunk pool, `buildChunk`, `scatterProps` (the pacing sections), `updateChunks` |
| `PLAYER` → `rider: geometry / rig / animation` | Board + rigged rider, two-bone IK, `poseRider` |
| `carve trail` / `snow spray` / `weather snow` / `speed lines` | The four particle-ish systems |
| `AUDIO` | WebAudio graph: looping wind + carve noise, one-shot whoosh/thud/chime |
| `INPUT` | Keyboard, multi-touch, DeviceOrientation permission flow and axis remap |
| `DAY / NIGHT + WEATHER` | Keyframe palette table, `applyDay`, weather state machine |
| `GAME STATE` | All mutable run state; crash / continue / leaderboard helpers |
| `SIMULATION` | `simulate` (the hot loop), pose, camera, trail, particles, HUD |
| `shareable card` | Canvas → JPEG export of the best run |
| main loop | `frame()` — orchestration and per-frame ordering |

## Reading order for a new session

1. `tuning` — the constants tell you the intended feel.
2. `simulate()` — the physics hot loop; everything else serves it.
3. `frame()` — the per-frame call order, which matters (see [[physics]]).

## Context log

### 2026-07-27 — Article created
Written because the generated codemap is empty by design for a single-file HTML
program. [[journal/2026-07-27-feel-rework]]
