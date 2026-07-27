# ALL DOWN HILL

**A 3D endless snowboarding game that ships as a single HTML file.** Three.js from a CDN, everything else generated at runtime — terrain, trees, rocks, sky, audio, the rider. No build step, no assets, no dependencies beyond one pinned module.

> **Scope:** a personal project and a deliberate constraint exercise — one file, procedural everything.
> **Status:** playable and deployed. Build mechanics, decisions and history live in [`knowledge/`](./knowledge/AGENTS.md).

**Play:** https://inkxel.github.io/all-down-hill/

---

## Why this exists

Alto's Adventure is a 2D side-scroller whose whole feel comes from one idea: your jump height is decided by *where on the terrain you leave the ground*, not by a button you hold. That idea should survive the move to 3D and a chase camera — this is the wager the project tests.

The single-file constraint is the second wager: that a complete 3D game with a day/night cycle, weather, physics and audio fits in something you can read top to bottom and deploy by pushing.

## What it does

1. **Endless procedural descent.** Infinite slope from layered simplex noise, chunked and recycled ahead of you. Nothing is allocated after startup.
2. **Carve and jump.** Steer laterally with momentum; press at a lip to launch, hold to rotate, release to freeze the angle. Land clean for a multiplier.
3. **Paced terrain.** Props follow a 350m rhythm — thread the trees, breathe, play on small kickers, commit to one full-width feature, land it.
4. **Tricks on two axes.** Straight kickers throw a backflip; kickers yawed 30° throw a barrel roll, judged on landing and worth double.
5. **A living scene.** A four-minute day/night cycle and four weather states cycling independently, driving fog, sky, light and particles.
6. **A bounded run.** Three continues per wipeout, then a local leaderboard with a shareable JPEG card.

## Controls

| | Desktop | Mobile |
|---|---|---|
| Carve | `A` / `D` or `←` / `→` | Tilt the device (or drag the lower half) |
| Jump | `SPACE` at the lip | Tap at the lip |
| Spin | Hold after launch, release to stop | Hold after launch |

## Architecture

One `index.html`: markup, CSS, and a single ES module. The program is sectioned with banner comments and indexed in [`knowledge/wiki/index-map.md`](./knowledge/wiki/index-map.md) — read that before grepping.

The load-bearing shape:

- **Terrain is a pure function.** `terrainHeight(x, z)` is queried directly by physics; the chunk meshes are a *rendering* of it, never the source of truth. See [`wiki/terrain.md`](./knowledge/wiki/terrain.md).
- **Simulation is one ordered function.** `simulate(dt)` — speed, steering, integration, vertical, collision. Order matters and is documented in [`wiki/physics.md`](./knowledge/wiki/physics.md).
- **The rider is rigged, not modelled.** A joint hierarchy with two-bone IK legs, animated entirely from game state. See [`wiki/rider.md`](./knowledge/wiki/rider.md).
- **Mood is data.** An eight-keyframe palette table drives every light, colour and fog value. See [`wiki/atmosphere.md`](./knowledge/wiki/atmosphere.md).

## Direction & principles

- **One file, or it doesn't ship.** Every dependency question resolves against this.
- **Feel over fidelity.** Physics constants exist to be tuned by playing, not derived.
- **Procedural means procedural.** No textures, no models, no audio samples.
- **Verify in a browser.** There is no unit-test surface; correctness claims come from driving the running game.
- **Effects are the product.** Particles, trails, shake, slow-mo and weather are the reason to build this. They are explicitly protected from complexity review.

## Key decisions

Stated here; reasoning lives in the linked records.

- One self-contained file, Three.js from CDN, no addons — [ADR](./knowledge/decisions/2026-07-26-single-file-cdn-three.md)
- Terrain height is analytic, not raycast — [ADR](./knowledge/decisions/2026-07-26-analytic-terrain-height.md)
- Fixed-slot instancing for props — [ADR](./knowledge/decisions/2026-07-26-fixed-slot-instancing.md)
- Alto's-style jump, no charge meter — [ADR](./knowledge/decisions/2026-07-27-altos-jump-model.md)
- Section-based pacing over uniform scatter — [ADR](./knowledge/decisions/2026-07-27-section-based-pacing.md)
- Three continues, then a leaderboard — [ADR](./knowledge/decisions/2026-07-27-three-continues-leaderboard.md)

## Roadmap

- **Phase 0 — done.** Complete game, deployed, reviewed for correctness and complexity.
- **Phase 1 — done.** Feel rework from play notes: jump model, pacing, camera, continue flow, rigged rider.
- **Phase 2 — current.** Tune from Tucker's notes on the rigged rider. Pose constants are provisional.
- **Phase 3 — parked.** Remaining rider treatments, roll-landing assist, breaking the fixed-period section rhythm. See [`wiki/roadmap.md`](./knowledge/wiki/roadmap.md).

## Repo layout

```
index.html                 the entire program
README.md                  this file
knowledge/
  AGENTS.md                orientation for any agent — read first
  wiki/                    curated reference per subsystem
  decisions/               ADRs
  journal/                 per-session build history
  research/                findings, comps, review ledgers
  wiki/roadmap.md          the parking lot
.githooks/post-commit      auto-journals every commit
```

## Context

Build mechanics, rationale and session history live in [`knowledge/`](./knowledge/AGENTS.md). Start with `knowledge/AGENTS.md`, then `wiki/index-map.md` to find your way around the source.
