---
type: Journal
date: 2026-07-27
session: landing-fix
description: Root-caused "landings are too precise" to a systemic dead zone where any hold past 90ms guaranteed a crash; fixed with a settle-to-nearest-turn assist.
status: shipped
related: [[physics]], [[decisions/2026-07-27-spin-settle-assist]]
---

# Landing fix — the rotation dead zone

## Context
Tucker's notes after playing the rigged-rider build: landings need "way too much
precision which makes it not fun," bumps feel great, but jumping off ramps
"would just crash the second I jumped."

Two complaints that sounded separate turned out to be one bug.

## What I measured before touching anything
Instrumented `land()` to log rotation, offset-from-upright, airtime and outcome,
then swept hold duration on plain terrain jumps. Every hold from 150 ms to 900 ms
crashed; only 0 ms and 80 ms landed.

The arithmetic behind it: `FLIP_RATE` 360°/s against a typical 0.85 s airtime tops
out near 300°. **A full rotation was not completable in a normal jump.** So the
window was: don't touch the button, or crash.

That explains both complaints at once — bumps felt great because he wasn't
holding; ramps felt broken because pressing at a lip and instinctively keeping
hold is the natural input.

## What changed
`settleSpin` — while airborne and not holding, ease toward the nearest whole
turn. Past 180° it carries you round and scores the trick; under it, straightens
you up. Tolerances 25°/40° → 45°/55° to cover the ease's tail.

Same bot input before and after: **24/24 landings crashed → 11/34**, tricks
landed 5 → 11.

## What was tried and abandoned
- **Widening the tolerance alone** — to cover the dead zone it would need to be ±180°, which is no fail state at all.
- **Slowing the rotation to fit the airtime** — looks sluggish and still leaves a dead zone either side of a full turn.

## An honest note on the ramp half
I also guarded `land()` against being called while `vy > 0`, on the theory that
skimming up a ramp face was being judged as a landing. **That branch never fired
in any test run** — `skims: 0` across long play runs. It is a correct defensive
guard and it stays, but I have no evidence it was the ramp culprit. The evidence
points entirely at the rotation dead zone. If ramp-specific instant crashes
survive this fix, that theory is still unproven and worth re-opening.

## Open threads
- [ ] Tucker to re-play and confirm the landing feel.
- [ ] 32% landing-crash rate under an adversarial bot (holds 450–650 ms then lands immediately, ignoring terrain). Unknown what a human rate looks like.

## Commit log

### 11:14 — 2749998
Document the landing fix: ADR, journal entry, wiki correction
files: knowledge/decisions/2026-07-27-spin-settle-assist.md, knowledge/decisions/index.md, knowledge/journal/2026-07-27-feel-rework.md, knowledge/journal/2026-07-27-landing-fix.md, knowledge/journal/index.md, knowledge/wiki/physics.md, knowledge/wiki/roadmap.md
