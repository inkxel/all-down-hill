---
type: Decision
date: 2026-07-27
description: Obstacles clipped in the air become launchers that link a trick chain, banked on the next clean landing; downed logs are grindable.
status: accepted
deciders: Tucker
related: [[physics]], [[terrain]]
---

# Decision: Trick chain — obstacles as launchers, logs as grinds

## Context
Tucker, after the landing fix started working: "would love to add a little bit of
trick stacking." Three specific ideas — bounce off a rock you land on, use the
top of a tree as a spring, and grind downed logs, jumping off before the log ends
or crashing.

The unifying idea is that a flight should be able to contain more than one trick.

## Decision
A `chain` counter, incremented by each in-air obstacle launch and by each 0.25s
of grinding, capped at 8. It banks on the next clean landing — raising the
multiplier and paying a bonus that scales with chain length, not just link count
— and is lost on a crash.

- **Rock crown** — bounce, returning 72% of downward speed with a 9 u/s floor.
- **Tree crown** — top 26% springs you at 12.5 u/s, an outright height gain.
- **Log** — jump on to lock to its line; a link every 0.25s; jump off before the end or crash. Riding into one at snow level is an obstacle.

Hitting a rock or tree at snow level still crashes, unchanged.

## Consequences
- **Positive:** obstacles become route-planning material rather than pure hazard, which is what makes stacking possible at all.
- **Positive:** logs restore the rhythm beat the removed ramps left empty, and reintroduce a commitment mechanic (you must exit before the end).
- **Negative:** the terrain is now more forgiving overall. A run that used to end on a rock can now continue and score.
- **Negative:** `chain` interacts with `mult`, which is already capped at 9, so long chains saturate quickly.
- **Neutral:** barrel rolls remain unreachable — nothing sets `spinAxis` while ramps are off.

## Two bugs this surfaced, both worth remembering
- **Point-in-time height tests miss fast crossings.** At 42 u/s the rider crosses an obstacle crown inside a single frame, so testing height only at the frame's end point read as "below the crown" and crashed instead of bouncing. Now the *band swept during the frame* (`hLo`..`hHi`) is tested.
- **A launcher with no cooldown compounds.** Contact persisted across frames, so a single rock re-launched every frame and stacked to 33 u/s — a 17m rocket. A 0.28s pass-through after any spring fixes it.

## Dissent / Alternatives Considered
- **Make every obstacle bounceable from any angle** — removes the fail state; the crown-only band is what keeps a mistimed approach fatal.
- **Score the chain immediately per link** — banking on landing is what makes a long flight a gamble rather than a guaranteed payout.
- **Grind by proximity rather than requiring a jump** — Tucker specified jumping onto it; auto-snapping would remove the approach skill.

## Sources
- [[journal/2026-07-27-trick-chain]]
