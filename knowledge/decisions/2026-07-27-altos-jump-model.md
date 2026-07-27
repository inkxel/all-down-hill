---
type: Decision
date: 2026-07-27
description: Replaced the hold-to-charge jump with Alto's model — a press adds a hop to existing surface velocity, and holding in the air rotates.
status: accepted
supersedes: charge-based jump from the original spec
deciders: Tucker
related: [[physics]], [[rider]]
---

# Decision: Alto's-style jump, no charge meter

## Context
The original spec called for hold-SPACE-to-charge with impulse scaling 8–22 over
0.8 s, then a *second* press-and-hold in the air to backflip. In play Tucker
found it clunky: two distinct holds for one manoeuvre, and jump height decoupled
from where you were on the terrain.

## Decision
Remove charge entirely.
- A press adds `JUMP_POP` (9.5) *on top of* whatever upward surface velocity the rider already carries. Hitting a lip at speed is what produces height.
- Continuing to hold in the air rotates — pitch normally, roll off an angled kicker.
- Releasing freezes the rotation where it is.

One press, one hold, one release for the whole trick.

## Consequences
- **Positive:** jump height becomes a function of *line choice and timing*, which is the actual skill in Alto's. Rolling terrain became fun to ride rather than something to get past.
- **Positive:** removes `charge`, `CHARGE_MAX`, `JUMP_MIN`, `JUMP_MAX` and the crouch-charge visual.
- **Negative:** you can no longer deliberately produce a big jump on flat ground; on a flat run `JUMP_POP` alone gives about 1.4 m.
- **Negative:** the input is now edge-triggered (`jumpPressed`), which has to be consumed exactly once per frame and reset — a held key must not re-fire.

## Dissent / Alternatives Considered
- **Keep charge but shorten it** — treats the symptom. The complaint was the two-hold structure, not the duration.
- **Charge only on flat ground, auto-pop on lips** — mode-dependent input, worse to explain and worse to feel.

## Sources
- Tucker's play notes. [[journal/2026-07-27-feel-rework]]
