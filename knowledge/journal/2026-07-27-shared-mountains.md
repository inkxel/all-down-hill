---
type: Journal
date: 2026-07-27
session: shared-mountains
description: Built the shared mountains arc — seeded terrain, ghost tracks carried in the URL, a top-5 ladder, a guaranteed clear start lane, and weather-driven music.
status: shipped
related: [[plans/2026-07-27-shared-mountains-design]], [[terrain]], [[wiki/audio]]
---

# Shared mountains

Built to [[plans/2026-07-27-shared-mountains-design]], in the spec's build order.

## Determinism first
Three places broke it and all three had to be fixed before anything else could
work: noise seeded by a load-time IIFE, prop scatter mixing only the chunk index,
and weather rolling bare `Math.random()`.

Weather is now keyed to **distance, not wall time** — two riders on one link
should meet the same storm at the same point on the mountain however fast they
ride it.

Verified: same seed reproduces terrain, props and weather byte-identically,
differs from another seed, and survives a page reload.

## Ghosts in the URL
One byte per sample — 7 bits of lateral position, 1 bit airborne — every 12m.
Round-trips byte-identically. Six riders on a 2400m run is 1.65KB.

The spec said 8m and ~2KB. Measuring showed 8m put six riders at **3.9KB**, past
the point anyone pastes a URL into a message, so the step widened to 12m. The
spec's estimate was wrong and the measurement corrected it.

## The clear start lane
Tucker, mid-build: he kept spawning inside or next to a tree and starting runs
with a crash. Sharper than it looks now — with seeded mountains that would be
*permanently* broken for everyone who opened that link.

Every mountain now opens with a 150m clear lane, 26m wide. Verified across 60
seeds: closest obstacle ahead in the lane is 125m of travel, ~7s at start speed.

This also forced a related fix: **Ride again now restarts at the top** of the same
mountain rather than continuing from the crash. Ghost lines are recorded from the
start, so a mid-mountain restart would have misaligned every one of them.

## Two mistakes worth recording
**A replacement hit the wrong occurrence.** `startMountain((Math.random()...))`
appeared twice, and my edit patched the one inside the *New mountain* button
handler instead of the load-time call — mangling the handler and silently leaving
shared links unadopted. The symptom was subtle: the title screen just didn't
mention the mountain. Caught only because the end-to-end share test asserted on
that text.

**A test read stale data and reported a false failure.** The determinism
fingerprint sampled `c.rx[0]` unconditionally, including for chunks with `rn = 0`
where that slot holds leftovers from a previous build. It reported
DETERMINISM FAILED on correct code. Fixed to only fingerprint slots in use.

Both cost a cycle each, and both were only distinguishable from real bugs by
looking at *what specifically* differed rather than trusting the pass/fail.

## Open threads
- [ ] Ghost line contrast is a guess (0.44 opacity on snow). Needs a human eye.
- [ ] A ghost is a trace, not a rival — no timing, so it cannot be raced. Deliberate; revisit only if racing is wanted.
- [ ] README hook can now honestly become "ride someone else's mountain and see where they fell". Not yet written.
- [ ] No mute control, still.

## Commit log
