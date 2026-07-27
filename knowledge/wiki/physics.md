---
name: physics
description: How the rider moves — surface following, the launch test, the Alto's-style jump, trick judging, and the two bugs that shaped all of it.
type: subsystem
created: 2026-07-27
last_updated: 2026-07-27
confidence: high
related: [[terrain]], [[rider]], [[decisions/2026-07-27-altos-jump-model]], [[decisions/2026-07-26-analytic-terrain-height]]
---

# Physics — movement, launch, tricks

## Frame order matters

`simulate(dt)` runs in a fixed order and several parts depend on it:

1. Forward speed target (ramps to `SPD_MAX` over `SPD_RAMP` seconds, modulated by local gradient, scaled by any landing boost).
2. Lateral steering — acceleration, friction when idle, soft out-of-bounds nudge, clamp.
3. Position integration.
4. **Vertical** — `const gy = groundAt(px, pz)` runs *first* in this block, which is what makes the next line valid.
5. Obstacle collision.

**`_rampSpin` is a side-effect of `rampRise`.** It is read immediately after the
first `groundAt(px, pz)` call and is only correct there — any later `groundAt`
call (the previous-position sample, the look-ahead sample) overwrites it. Do not
move that read.

## Grounded: surface following

When grounded, `y` snaps to `gy` and `vy` is set to the *surface velocity* —
how fast the ground is rising or falling underneath you, `(gy - prevGy) / dt`.

This is clamped to `[-55, SURF_VY_MAX]`. The clamp is load-bearing, not defensive
tidiness: see the catapult below.

## Leaving the ground

Two paths, both setting `spinAxis`:

- **Look-ahead launch.** If a free-fall arc projected 0.10 s forward would clear the terrain ahead by more than 0.28, you go airborne. This is what makes crests and kicker lips throw you naturally.
- **Jump press.** A press adds `JUMP_POP` on top of whatever upward surface velocity you already carry. There is no charge — timing the lip *is* the mechanic. See [[decisions/2026-07-27-altos-jump-model]].

## Spin arming

An angled kicker is crossed *diagonally*, so you frequently exit its side
footprint a frame or two before the launch fires — at which point `_rampSpin` has
already reset to 0. `armedSpin` / `armedT` latch the last kicker's spin axis for
0.30 s so both launch paths see it.

Airborne, holding the jump input rotates: pitch (backflip) when `spinAxis === 0`,
roll (barrel roll) otherwise. Releasing freezes the angle where it is.

## Landing judgement

Both axes are wrapped to ±180° and tested: pitch within `LAND_TOL` (25°), roll
within `ROLL_TOL` (40°, deliberately forgiving because rolls are harder to read).
Failing either is a crash. Rolls score double.

## The two bugs worth remembering

**The catapult.** Stepping *sideways* onto a ramp moved ground height the full
1.89 m in a single frame. Surface velocity divides by dt, so that read as ≈58 u/s
of lift and threw the rider 73 m into the air. Fixed twice over — the ramp edge
now tapers laterally so there is no cliff to step onto, *and* surface lift is
clamped so any future discontinuity is contained. Peak air went 72.9 m → 6.1 m.

**Inert ramps.** `THREE.MathUtils.smoothstep` is `(x, min, max)` and
early-returns 0 when `x <= min` — it cannot be used inverted. The first version
of the edge taper was written as `smoothstep(adx, hw, hw * 0.62)`, which returned
0 at *every* point on a ramp. Ramps were completely non-functional and it was
invisible in play because terrain rollers still launched you. If a ramp ever
stops working, check that expression first.

## Context log

### 2026-07-27 — Article created
State after the feel rework. [[journal/2026-07-27-feel-rework]]
