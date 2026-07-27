---
type: Decision
date: 2026-07-26
description: Physics queries the noise function directly instead of raycasting the terrain mesh; the mesh is a rendering of the function, not the source of truth.
status: accepted
deciders: Tucker
related: [[terrain]], [[physics]]
---

# Decision: Terrain height is analytic, not raycast

## Context
The rider needs ground height and surface normal every frame, plus a look-ahead
sample for the launch test. The obvious approach is raycasting the terrain mesh.

## Decision
`terrainHeight(x, z)` is a pure function of layered simplex noise. Physics calls
it directly. The chunk meshes are a *rendering* of that function — they never
feed back into simulation. Normals come from finite differences of the same
function.

## Consequences
- **Positive:** no raycaster, no BVH, no per-frame allocation. About 12 noise evaluations per frame total.
- **Positive:** ground height is defined everywhere, including past the rendered chunks, so look-ahead never falls off the edge of the world.
- **Positive:** mesh resolution becomes purely a visual knob — changing it cannot change physics.
- **Negative:** the mesh and the function can silently disagree if a chunk build ever diverges from `terrainHeight`.
- **Negative:** props that alter the surface (ramps) must be added analytically too, via `rampRise` — which is where the catapult bug came from.

## Dissent / Alternatives Considered
- **Raycast the chunk meshes** — allocates, needs a spatial structure to be fast, and gives wrong answers at chunk seams the frame a chunk recycles.
- **Store a heightfield array per chunk** — would make physics and render agree by construction, but adds a second source of truth to keep in sync and doesn't extend past the rendered region.

## Sources
- [[journal/2026-07-26-build-and-review]]
