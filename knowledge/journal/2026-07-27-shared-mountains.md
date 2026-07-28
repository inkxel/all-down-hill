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

## Feel pass — Tucker's six
1. **Trails sinking into the snow.** Not z-fighting. The rendered snow is a 2m triangulation of the height function; across a hollow the flat triangle sits *above* the true surface, so a line drawn at the analytic height ends up under the mesh. Both ghost and carve trails now take a local maximum and lift clear. **Fix the cause, not the symptom — raising the offset alone would have papered over it.**
2. **Yard sale.** Gear strewn at every crash — ghosts' and your own, persisting across a re-ride. Cosmetic, never collidable.
3. **Log snapping.** My overcorrection, twice over: the assist fired from **8m** off the line. Now 2.6m, and holding no longer triggers it at all. Measured at 2m off the line: tap grabs the log 4/6, hold **0/6**.
4. **Trick variety.** Grab default, up front, down back. Payout 1.30 fresh, 0.35 after three repeats, 1.60 switching.
5. **Roll from carving** — replaces the ramp dependency entirely and is a better mechanic: you are already banked.
6. **Doubles were impossible, not hard.** The anti-catapult cap allowed 1.5s of air = 540°. Same shape as the landing dead zone: a thing that felt "rare" was arithmetically unreachable.

### The tap/hold problem worth remembering
Intent could not be read at the instant of the press, and delaying the jump to
find out would have put latency on *every* jump in the game. Solution: take off
normally, then retarget on an early release when only ~150ms has passed. **When
an input is ambiguous, act on the common case immediately and correct on the
distinguishing signal.**

## Idle title screen was cooking the CPU
Tucker left the page on the title for 20 minutes and came back to hot fans. Not a
leak — heap and node count measured flat. The loop was rendering the **entire 3D
scene into a canvas at opacity 0**, plus a full-DPR 2D repaint, at 60fps, forever.
All invisible.

Title now renders nothing 3D, repaints 2D at 30fps, backs off to 6fps after 90s
untouched, and a hidden tab does nothing. Task time per frame 0.93ms → 0.40ms
even under headless throttling, which understates it since the eliminated 3D work
is mostly GPU.

**Generalisable:** "it's slow" reflexively suggests a leak. Measure first — flat
heap pointed straight at sustained-but-bounded waste instead.

## Open threads
- [ ] Ghost line contrast is a guess (0.44 opacity on snow). Needs a human eye.
- [x] ~~Second Codex pass pending~~ — completed 2026-07-28, see [[journal/2026-07-28-codex-pass]].
- [ ] A ghost is a trace, not a rival — no timing, so it cannot be raced. Deliberate; revisit only if racing is wanted.
- [ ] README hook can now honestly become "ride someone else's mountain and see where they fell". Not yet written.
- [ ] No mute control, still.

## Commit log

### 20:40 — 2a9eb17
Document the shared mountains build
files: knowledge/journal/2026-07-27-music-and-dismount.md, knowledge/journal/2026-07-27-shared-mountains.md, knowledge/journal/index.md, knowledge/plans/2026-07-27-shared-mountains-design.md

### 21:43 — 93541a1
Trick variety, yard sale, tighter log snap, reachable doubles
files: index.html

### 21:52 — e449468
Stop burning CPU on an idle title screen
files: index.html
