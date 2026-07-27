---
type: Decision
date: 2026-07-26
description: Trail and spray vertices are stored relative to a periodically re-based origin object, because the run travels tens of thousands of units from world zero.
status: accepted
deciders: Tucker
related: [[physics]], [[index-map]]
---

# Decision: Re-base trail and spray origins for float32 precision

## Context
The rider descends indefinitely — a long run reaches z ≈ −30,000 and y ≈ −11,000.
Vertex buffers are float32, where one ulp at 32,768 is about 0.004 units. Storing
absolute world positions in a buffer that far out produces visible jitter.

## Decision
Trail and spray buffers hold positions *relative* to an origin `Object3D`. When
the player drifts more than 300 units from that origin, `rebase()` shifts every
stored position by the delta and moves the object. One shared helper covers both.

## Consequences
- **Positive:** stored coordinates stay small, so precision stays constant regardless of run length.
- **Positive:** re-basing is rare (every ~300 units) and bounded.
- **Negative:** an extra indirection when writing particles — emitters must subtract the origin.
- **Neutral:** terrain chunks don't need this; their vertices are already local and only `mesh.position` is large, which three.js composes in float64 on the CPU.

## Dissent / Alternatives Considered
- **Re-base every frame** — ~2,700 float ops per frame, genuinely negligible, but pointless work when a 300-unit threshold gives the same result.
- **Do nothing** — measured error is ~0.01 units at 30 k, which is invisible for props but not for a thin ribbon lying on the snow.
- **Wrap world position back to origin periodically** — would require moving the player, camera, chunks and all props in lockstep; far more invasive.

## Sources
- [[journal/2026-07-26-build-and-review]]
