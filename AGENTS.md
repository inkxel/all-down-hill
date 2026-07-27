# ALL DOWN HILL — agent instructions

A 3D endless snowboarding game that ships as one self-contained `index.html`.
Three.js from a pinned CDN module; everything else is generated at runtime.

**Read [`knowledge/AGENTS.md`](./knowledge/AGENTS.md) before doing architectural
work.** It carries the orientation, the three documentation rules and the entry
formats. Then read [`knowledge/wiki/index-map.md`](./knowledge/wiki/index-map.md)
to navigate the source — the generated `_codemap.md` is empty for this repo by
design, because the whole program lives in one inline `<script>`.

## Hard constraints

- **One file.** No `src/`, no build step, no bundler, no second dependency. Three.js addons count as a second dependency.
- **Everything procedural.** No textures, no models, no audio samples.
- **Verify in a browser.** There is no unit-test surface. Correctness claims come from driving the running game with Playwright, not from reading the diff.
- **Effects are the product.** Particles, carve trails, screen shake, slow-mo, speed lines, the day/night cycle, weather and audio are deliberate. Do not simplify them away in the name of tidiness.

## Before changing physics

Read [`knowledge/wiki/physics.md`](./knowledge/wiki/physics.md) first. Two traps
are documented there that have already cost a session each: the frame-order
dependency on `_rampSpin`, and `MathUtils.smoothstep` being `(x, min, max)` and
therefore un-invertible.
