---
type: Journal
date: 2026-07-27
session: feel-rework
description: Reworked jump, pacing, camera and the wipeout flow from Tucker's play notes; root-caused a 73m catapult; shipped the rigged rider; scaffolded this knowledge layer.
status: shipped
related: [[physics]], [[terrain]], [[rider]]
---

# Feel rework — Tucker's first play notes

## Context
Tucker played the build and returned six notes: the charge jump felt clunky, the
camera was too low, everything was too frequent and the ramps redundant, a glitch
launched the rider off-screen, the character looked bad, and he wanted a
keep-going prompt with a downloadable leaderboard.

Three questions were genuinely blocking and got asked up front: continue economy
(answer: **3 per run**), barrel-roll landing strictness (answer: **judged, but
generous at 40°**), and rider art direction (answer: **combine rounded low-poly
with graphic silhouette, show 5 comps first**).

## What changed
- Jump reworked to Alto's model — [[decisions/2026-07-27-altos-jump-model]]
- Pacing reworked into sections — [[decisions/2026-07-27-section-based-pacing]]
- Continue + leaderboard + JPEG card — [[decisions/2026-07-27-three-continues-leaderboard]]
- Camera raised and pitched further down the hill.
- Angled kickers now throw barrel rolls; roll is a second landing axis.
- Box rider replaced with the rigged one — [[rider]]

## The catapult — reproduced, not guessed
Tucker reported the rider flying "way high up in the air" in the first minute.
Rather than tune gravity, instrumented the launch path and logged every event:

```
{"k":"launch","vy":60.1,"fwd":18.8,"rise":1.89,"prevRise":0,"dt":0.0323}
```

`prevRise: 0 → rise: 1.89` in one frame. Stepping *sideways* onto a ramp moved
ground height the full kicker height instantly; surface velocity divides by dt,
so `1.89 / 0.032 = 58 u/s` of lift → 73 m of air.

Fixed at both levels: taper the ramp's lateral edges so there is no cliff, and
clamp surface lift so any future discontinuity is contained. Peak air 72.9 m →
6.1 m.

## The bug the fix exposed
The first edge taper was written `smoothstep(adx, hw, hw * 0.62)` — inverted,
carrying over intuition from the hand-rolled `(edge0, edge1, x)` helper that
[[journal/2026-07-26-build-and-review]] had just replaced. `MathUtils.smoothstep`
early-returns 0 when `x <= min`, so it returned 0 at **every** point on a ramp.

**Ramps had been completely inert.** Invisible in play because terrain rollers
still launch you — which is very likely why Tucker's note said ramps felt
"redundant." A feel complaint was actually a dead code path.

Caught only because the barrel-roll test asserted on `onRamp` frame counts rather
than on whether the game looked right.

## What was tried and abandoned
- **Reading `_rampSpin` at launch via a fresh `rampRise` call** — angled kickers get crossed diagonally, so you often exit the side footprint a frame or two before the launch fires and the spin has already reset. Replaced with a 0.30 s armed-spin latch.
- **A flat ribbon scarf** — vanished edge-on from the chase camera. Rebuilt as a swept twisting tube.
- **Parenting the scarf to the chest** — torso twist swung it sideways across the silhouette. Moved to the rig root.
- **Blind numeric pose tweaking** — burned several iterations. Switched to rendering the rig from four fixed orthogonal angles, which made every problem obvious immediately.

## Open threads
- [ ] Tucker to play the rigged rider and give feel notes — pose constants are provisional.
- [ ] Rider comps 2–5 remain unbuilt; only Ink shipped.
- [ ] Section rhythm is fixed-period and will feel loopy over a long session.

## Related
- [[research/2026-07-rider-comps]]
