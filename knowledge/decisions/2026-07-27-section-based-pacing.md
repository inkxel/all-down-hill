---
type: Decision
date: 2026-07-27
description: Props follow a repeating 350m rhythm — slalom, run-up, kickers, one feature, landing — instead of uniform random scatter.
status: accepted
supersedes: uniform per-chunk random scatter
deciders: Tucker
related: [[terrain]]
---

# Decision: Section-based pacing instead of uniform scatter

## Context
The original generator scattered trees, rocks and ramps uniformly per chunk with
density rising by distance. Tucker's note: everything felt "a little too
frequent," ramps felt "redundant," and there was no run-up to anything. He wanted
a pattern — thread the trees, then a straightaway, building to a big feature.

## Decision
`SECTIONS` is a seven-entry cycle, one entry per 50 m chunk (350 m total):
slalom → tighter slalom → run-up → small kickers → run-up → one full-width
feature → landing. Per-section counts, with a distance-based load multiplier
layered on top. Run-up sections push trees out to the treeline so the middle is
genuinely clear.

Every other feature kicker is yawed 30°, which throws a barrel roll instead of a
backflip.

## Consequences
- **Positive:** runs went from ~15 m before a wipeout to 400 m+. There is now somewhere to breathe and something to build toward.
- **Positive:** the big feature is unmissable (15 units wide, near centre) so it reads as an event.
- **Negative:** the rhythm is fixed-period, so a long session will feel the loop. Mitigated by the density ramp but not solved.
- **Neutral:** difficulty still scales with distance; the sections change *where* things are, not *how many* overall.

## Dissent / Alternatives Considered
- **Just lower the global density** — Tucker explicitly asked for a *pattern*, not simply less stuff. Uniform-but-sparse still has no build-up.
- **Fully procedural pacing (noise-driven density field)** — smoother, but you lose the authored "here comes the big one" beat that was the point.
- **Hand-authored set pieces** — best feel, incompatible with infinite generation.

## Sources
- Tucker's play notes. [[journal/2026-07-27-feel-rework]]
