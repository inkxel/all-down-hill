---
type: Journal
date: 2026-07-27
session: tilt-and-hop
description: Fixed tilt steering at the signal and control-model level, added a log hop assist, and rewrote the README as a game intro.
status: shipped
related: [[mobile]], [[decisions/2026-07-27-tilt-gravity-projection]]
---

# Tilt, hop assist, README

## Context
Three asks: a proximity hop so logs are landable, a real fix for tilt steering
(Tucker asked whether prior art existed on GitHub), and a README that reads as a
game rather than a project write-up. Mid-session he added: don't reference
Alto's Adventure in the README.

## Tilt — researched rather than tuned
The previous pass softened the response curve and it still wasn't controllable,
which is the signal that the problem is not gain. Two real causes:

1. **`gamma` is the wrong signal.** One Euler angle of a Z-X'-Y'' triple: it under-reads with pitch and is degenerate near vertical.
2. **Acceleration is the wrong control model** for an absolute-position device. It made lateral position a double integrator of tilt.

Fixed both. [[decisions/2026-07-27-tilt-gravity-projection]]

Measured, same wrist roll at three hold angles:

| roll | before (any pitch) | now @0° | now @30° | now @60° |
|---|---|---|---|---|
| 10° | 10 | 10.2 | 10.2 | 10.2 |
| 20° | 20 | 21.3 | 21.3 | 21.3 |
| 30° | 30 | 35.3 | 35.3 | 33.7 |

## What I got wrong on the way
First attempt at pitch-independence used the in-plane roll angle (`atan2`) with a
magnitude threshold to fall back to the projection when flat. It produced a
**discontinuity from 15° to 90°** at the crossover — measurably worse than what it
replaced. Backed out and solved it as a gain problem instead (divide by flatness,
with a floor), which is both simpler and continuous.

Also burned several cycles on the test harness rather than the game: rebuilt the
debug build piecemeal three times and each rebuild dropped hooks the previous
tests depended on, producing convincing all-zero results twice that were pure
harness failure. Consolidated into one `mkharness.py`. **Lesson: a failing
measurement is not evidence until the harness itself is verified.**

## Log hop
A jump taken within 15m of a log now solves the vertical speed that lands on the
deck rather than throwing a full jump, aims a quarter of the way along it, and
eases laterally toward its line while airborne — player input still overrides.
Measured 83% when roughly lined up, falling off with lateral distance. Deliberately
an assist, not a magnet.

## README
Rewritten as a game intro: key art, gameplay GIF, screenshots, and a how-to-play
that leads on the three mechanics that need explaining. Project detail moved to
`knowledge/` and linked. Media generated from the running game and committed
under `media/` (1.3 MB total).

Per Tucker: no reference to Alto's Adventure. Note that the ADRs and older journal
entries still reference it as the design touchstone — that instruction was scoped
to the README, and the build history is left accurate.

## Open threads
- [ ] Tucker to judge tilt on a real phone. `TILT_FLATTEN` (0.45) and `TILT_FULL` (28) are the two knobs.
- [ ] Two control models now coexist (keyboard = acceleration, tilt = velocity). Remember when tuning either.
- [ ] Hop assist success unmeasured for a human aiming deliberately.

## Commit log
