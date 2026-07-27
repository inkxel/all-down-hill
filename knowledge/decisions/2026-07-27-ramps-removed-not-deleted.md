---
type: Decision
date: 2026-07-27
description: Ramp placement is switched off behind a flag; every physics path that makes ramps work is kept intact for a future redesign.
status: accepted
deciders: Tucker
related: [[terrain]], [[physics]], [[decisions/2026-07-27-section-based-pacing]]
---

# Decision: Ramps disabled, machinery retained

## Context
Tucker: "remove the ramps entirely because at this point they just aren't working
the way I want and they don't look as good as the rest of the scene. don't remove
the physics around it or the barrel roll because I think we can find a way to
build those back in but I just need to design it out a little more."

An explicit instruction to disable the feature without losing the capability.

## Decision
`RAMPS_ENABLED = false` gates prop placement only. Retained and still correct:
- `rampRise()` including the lateral edge taper and its rotated footprint
- `_rampSpin` / `armedSpin` / `armedT` spin arming
- barrel-roll rotation on the roll axis and its landing judgement
- `settleSpin` (which is not ramp-specific and still governs every trick)
- the ramp geometry builder and its InstancedMesh

`SECTIONS` keeps its non-zero `ramps` counts so the intended rhythm is still
legible in the data; only the gate suppresses them.

## Consequences
- **Positive:** re-enabling is a one-line change, and the rhythm design survives.
- **Negative:** barrel rolls are now unreachable in play — `spinAxis` is only ever set by an angled kicker. The code path is live but dead in practice.
- **Negative:** the section cycle loses its climax. What was "build up to one full-width feature" is now build-up to nothing.
- **Neutral:** the ramp InstancedMesh still allocates its instances at startup; all are zero-scale, costing nothing to draw.

## Dissent / Alternatives Considered
- **Delete the ramp code outright** — explicitly rejected by Tucker; he intends to redesign rather than abandon.
- **Keep ramps but restyle them** — the complaint was as much about how they play as how they look, and he wants to design the replacement himself first.

## Sources
- [[journal/2026-07-27-mobile-and-ramps]]
