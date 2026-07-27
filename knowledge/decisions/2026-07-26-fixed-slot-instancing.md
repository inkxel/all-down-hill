---
type: Decision
date: 2026-07-26
description: Each chunk owns a fixed instance-index range in each InstancedMesh; unused slots get a zero-scale matrix. No free list, no allocation.
status: accepted
deciders: Tucker
related: [[terrain]]
---

# Decision: Fixed-slot InstancedMesh allocation for props

## Context
Trees, rocks and ramps need to be instanced for draw-call count, but chunks
recycle constantly and each chunk holds a different number of props.

## Decision
Chunk `i` owns instance indices `[i * MAX, (i+1) * MAX)` in each `InstancedMesh`.
On rebuild, the chunk fills its own range and writes a zero-scale matrix to any
unused slot. `frustumCulled = false` on all three meshes.

## Consequences
- **Positive:** no allocator, no free list, no fragmentation. Recycling is a bounded loop over a known range.
- **Positive:** capacity is provably bounded — `N_CHUNKS × MAX_*` instances, allocated once.
- **Negative:** wastes slots when a chunk is sparse. At 9 × 26 tree slots that is a few hundred zero-scale matrices, which cost nothing to rasterize.
- **Neutral:** `frustumCulled = false` is required, not optional — `InstancedMesh` caches its bounding sphere on first use and would go stale after recycling, culling live props.

## Dissent / Alternatives Considered
- **A shared free list across chunks** — real allocator complexity and fragmentation for the sake of a few hundred unused matrices.
- **One InstancedMesh per chunk** — 27 draw calls instead of 3, and 27 buffers to manage.

## Sources
- [[journal/2026-07-26-build-and-review]]
