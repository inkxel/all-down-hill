---
type: Decision
date: 2026-07-27
description: A released spin eases to the nearest whole turn instead of freezing where it is; landing tolerances widen to 45/55 degrees.
status: accepted
supersedes: [[decisions/2026-07-27-altos-jump-model]]'s "releasing freezes the angle", and the 25/40 degree tolerances
deciders: Tucker
related: [[physics]]
---

# Decision: Ease a released spin to the nearest whole turn

## Context
Tucker played the rework and reported landings felt "way too precise which makes
it not fun," plus jumping off ramps that "would just crash the second I jumped."

Measured before changing anything — landing outcome against hold duration:

| hold | rotation at touchdown | result |
|---|---|---|
| 0 ms | 0° | clean |
| 80 ms | 21° | clean |
| 150 ms | 45° | **crash** |
| 450 ms | 156° | **crash** |
| 900 ms | 189° | **crash** |

Any hold past ~90 ms was a guaranteed crash. Worse, `FLIP_RATE` is 360°/s against
a typical 0.85 s airtime, so a full 360° rotation *could not be completed in a
normal jump at all*. The only safe strategy was never to press the button — which
is exactly why terrain bumps felt good (no hold) and ramps felt broken (instinctive
hold on press).

## Decision
While airborne and **not** holding, ease both rotations toward the nearest
multiple of 360° (`settleSpin`, clamped 240–900°/s).

- Past a half turn it carries you round to a full one and awards the trick.
- Under a half turn it straightens you up.
- Holding all the way into the ground still lands you wherever you are, so greedy spinning keeps its risk.

Tolerances widened from 25°/40° to **45° pitch / 55° roll** to cover the ease's
tail when you release late.

## Consequences
- **Positive:** the commit threshold becomes a readable ~0.5 s hold. Same bot input before and after: 24/24 landings crashed → 11/34, tricks landed 5 → 11.
- **Positive:** partial rotations resolve themselves, so a mistimed release is a missed trick rather than a wipeout.
- **Negative:** you get credit for a rotation you didn't quite finish — past 180° the assist completes it. Deliberate generosity; it is what makes the trick feel landable.
- **Neutral:** `FLIP_RATE` / `ROLL_RATE` unchanged. The assist, not the rate, is what made rotations achievable.

## Dissent / Alternatives Considered
- **Just widen the tolerance** — would need a ±180° window to cover the dead zone, which removes the fail state entirely.
- **Slow the rotation rate so a flip fits the airtime** — makes the flip look sluggish and still leaves a dead zone either side of 360°.
- **Increase airtime instead** — changes the whole feel of the terrain to fix a trick-input problem.

## Sources
- Tucker's play notes; measured with an instrumented build before any change was made. [[journal/2026-07-27-landing-fix]]
