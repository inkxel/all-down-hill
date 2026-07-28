---
type: Decision
date: 2026-07-27
description: Tilt steering derives from gravity projected into the screen frame and pitch-compensated, and drives target lateral speed rather than acceleration.
status: accepted
supersedes: raw-gamma tilt in [[decisions/2026-07-27-mobile-input-ungated]]
deciders: Tucker
related: [[mobile]], [[physics]]
---

# Decision: Gravity-projected, pitch-compensated tilt driving velocity

## Context
Tucker, after the sensitivity curve landed: still "really having trouble
controlling direction accurately" on mobile. He asked whether an existing
steering spec or library on GitHub could be used.

Two independent problems, found by researching prior art rather than tuning
further:

**The signal was wrong.** `gamma` is one Euler angle of a W3C Z-X'-Y'' triple.
It under-reads as the phone pitches — the same wrist roll produced steadily less
steering the steeper you held the device — and is degenerate near `beta = ±90`
(gimbal lock).

**The control model was wrong.** Tilt fed acceleration, making lateral position a
double integrator of the input. That is the hardest possible thing to place
accurately, independent of any sensitivity setting.

## Decision
**Signal.** Project gravity into the screen frame — the screen-adjusted approach
[Full-Tilt](https://github.com/adtile/Full-Tilt) and
[gyronorm.js](https://github.com/dorukeker/gyronorm.js) both take — then divide by
how flat the phone is so the gain is pitch-independent:

```
gravity (device)   = (cosB·sinG, −sinB, −cosB·cosG)
screen right       = ( cosA, −sinA, 0)
tilt = asin( (g · right) / max(0.45, |g_z|) )
```

The `0.45` floor stops the compensation exploding as the phone approaches
vertical.

**Control model.** Tilt is an absolute-position device, so it sets a *target
lateral speed* which `vx` damps toward. The keyboard is a momentary device and
keeps the acceleration model.

## Consequences
- **Positive:** measured response is now identical at 0°, 30° and 60° of pitch (5.0 / 10.2 / 15.5 / 21.3 / 27.8 for the same wrist rolls) where before it fell away with pitch.
- **Positive:** velocity control makes the rider directly placeable instead of something you fly.
- **Negative:** two control models now coexist. Keyboard and tilt feel measurably different, which is intended but is a thing to remember when tuning either.
- **Negative:** near-vertical holds are deliberately attenuated by the floor rather than compensated. A degenerate posture, but not corrected.
- **Neutral:** the expo curve and smoothing from the previous pass still apply, now on top of a better-conditioned signal.

## Dissent / Alternatives Considered
- **Vendor Full-Tilt or gyronorm.js** — both solve the general orientation problem with quaternions and matrices. Rejected: a second dependency violates [[decisions/2026-07-26-single-file-cdn-three]], and the game needs one scalar, not a full orientation stack. The *approach* was taken; the library was not.
- **In-plane roll angle (`atan2`) instead of the compensated projection** — genuinely pitch-invariant, but degenerate when the phone is flat, and blending the two produced a discontinuity from 15° to 90° at the crossover. Tried and measured; rejected.
- **Keep tuning sensitivity** — the previous pass already did that and Tucker still could not steer accurately, which is what pointed at the model rather than the gain.

## Sources
- [[journal/2026-07-27-tilt-and-hop]]
