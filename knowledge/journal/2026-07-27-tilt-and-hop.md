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

## Second pass — the bug behind the bug

Tucker: tilt is good on the first run but "every time I crash and choose to keep
going it gets messed up... veering hard left even if the phone is flat." He noted
this predated the tilt rework — it was a separate bug all along.

Root cause found by reading, not guessing: `respawn()` set `needBaseline = true`,
so the **next** sensor event captured neutral. On a device streaming at ~60Hz that
event lands ~16ms after the player's thumb leaves the Keep going button — while
the phone is still tilted from reaching for it. That tilted angle became "flat",
shifting the entire range.

Neutral is now relearned continuously while any panel is up and for 0.6s after
resuming, with steering suppressed until it settles.

| | phone flat after continue | then tilt 20° |
|---|---|---|
| before | **−0.733** | 0.000 |
| after | 0.000 | 0.590 |

**My first attempt at this test did not reproduce the bug** — it dispatched
orientation events only on demand, so the first event after respawn was already
flat and both builds looked fine. Only after emulating a continuous 60Hz stream
did the pre-fix build show the veer. A test that cannot fail is not evidence.

## Logs, take two
Tucker had still never landed one, and crashed when he got close. Widened the
grind capture, extended the hop assist to 8m off the line with a stronger pull,
put logs in five of seven section types instead of two, and made clipping the
side at snow level non-fatal — only a square hit kills. 26/26 landed in test
across offsets out to 4m.

## README
Dropped the GIF at Tucker's call — too fast to read; the stills do the job.
Removed the section pointing readers at `knowledge/`: he does not want the build
history promoted on a public repo, and flagged that he may gitignore or remove it
entirely later. **Not acted on** — the layer stays as-is until he says so.

## Open threads
- [ ] Tucker to judge tilt on a real phone. `TILT_FLATTEN` (0.45) and `TILT_FULL` (28) are the two knobs.
- [ ] Two control models now coexist (keyboard = acceleration, tilt = velocity). Remember when tuning either.
- [ ] Hop assist success unmeasured for a human aiming deliberately; capture may now be too sticky.
- [ ] Tucker may later gitignore or delete `knowledge/` for this public repo. Not done; his call.

## Commit log

### 18:21 — cbe7131
Document the tilt rework and hop assist
files: knowledge/decisions/2026-07-27-tilt-gravity-projection.md, knowledge/decisions/index.md, knowledge/journal/2026-07-27-tilt-and-hop.md, knowledge/journal/2026-07-27-trick-chain.md, knowledge/journal/index.md, knowledge/wiki/mobile.md

### 18:44 — 2d24ef8
Fix tilt neutral after a continue; make logs landable; README pass
files: README.md, index.html, knowledge/journal/2026-07-27-tilt-and-hop.md, media/gameplay.gif
