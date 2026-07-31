---
type: Journal
date: 2026-07-31
session: mute-and-readme
status: shipped
related: [[wiki/audio]], [[wiki/roadmap]], [[journal/2026-07-27-shared-mountains]]
description: Closed the two oldest open threads — a mute control the iOS silent-switch bypass had made overdue, and a README that never mentioned shared mountains.
---

# Mute control + the README hook

## Context

Asked what actually needed working on. Nothing was in flight — clean tree,
branch level with `main`, and **zero GitHub issues, open or closed**. Two items
had been sitting in journal open threads since 2026-07-27, both flagged twice and
never picked up:

- *"No mute control — now more pressing, since the game overrides the iOS silent
  switch."* [[journal/2026-07-27-music-and-dismount]]
- *"README hook can now honestly become 'ride someone else's mountain and see
  where they fell'. Not yet written."* [[journal/2026-07-27-shared-mountains]]

## What changed

- Paths touched: `index.html`, `README.md`
- Subsystems affected: [[wiki/audio]]
- Behavior shipped: a persistent mute; a README that documents the shared-mountains arc

## Mute belongs on `master`, and nowhere else

The obvious-looking places are all wrong. `musicGain` is rewritten every frame by
`updateMusic` (`musicGain.gain.value = MUSIC_GAIN * musicDuck`) and `windGain` /
`carveGain` every frame by `updateAudio`. Zero any of them and the next frame
puts it back.

`master` is the only node every voice routes through *and* the only one nothing
writes to per frame. That makes it the single correct choke point, and the test
asserts exactly that property rather than just "it went quiet":

> master held at 0 across 24 further per-frame gain writes.

**Generalisable:** in a graph where a render/update loop owns most parameters,
the mutable-by-the-loop nodes are not candidates for state the user sets. Find
the node the loop never touches.

## Muting also stops the iOS session override

The whole reason this thread was marked "more pressing" is that `unmuteIOS()`
deliberately promotes the audio session so Web Audio survives the ring/silent
switch ([[wiki/audio]]). Silencing the graph while leaving the silent element
looping would keep that override in place for a game making no sound — the exact
override the player just asked us to stop doing. So `setMuted(true)` pauses
`silentEl`, and unmuting re-arms it through `initAudio()`.

## Two things the browser caught that reading the diff would not

**The hint text ran under the toggle.** On any phone-width screen the bottom hint
(`DRAG TO CARVE — TAP TO JUMP, HOLD TO SPIN`) is nearly full width, and the
toggle sat straight on top of the last two words. Invisible on desktop, obvious
in a 375px screenshot. Fixed by giving the hint's bottom offset a shared
`--hint-bottom` custom property and anchoring the toggle to *that* rather than to
a guessed pixel value — move the hint and the toggle follows.

**`AudioParam` is float32-backed.** `master.gain.value = 0.85` reads back as
`0.8500000238418579`, so the first test run failed three assertions on an exact
`=== 0.85`. The product was correct and the test was wrong. Worth remembering
before trusting any exact equality against a Web Audio parameter.

## Verifying without the CDN

This sandbox's network policy blocks `cdn.jsdelivr.net`, so the pinned Three.js
module 404s and the entire program silently never executes — every DOM assertion
still "passes" against static HTML while nothing is actually running. `npm` is
reachable, and the npm artifact is the same file the CDN serves, so the harness
serves a copy of `index.html` with only the import line rewritten.

Note that `page.route()` did **not** intercept the module import in this Chromium
build; two attempts (glob and regex) were silently bypassed. Rewriting the import
in a served copy worked.

> A test suite that reports 9/27 with the module dead is more dangerous than one
> that errors out. Check that the program *ran* before trusting any green.

27/27 across desktop and iPhone emulation: mute holds under per-frame writes,
unmute restores full gain, a persisted mute applies when audio first inits, `M`
on the title mutes without dropping into the run, and a tap on the toggle
neither starts the run nor registers as a jump.

## Stale documentation found along the way

Not fixed blind — each was checked against the source first:

- **Barrel rolls are reachable.** [[wiki/roadmap]] and two journals say they are
  unreachable until ramps return. `spinAxis` is set from carve speed at
  `index.html:2902` and `:2920` — the feel pass replaced the ramp dependency and
  nothing updated the roadmap.
- **Pause exists.** The roadmap lists "No pause" as deliberate. `pauseGame()`,
  `resumeGame()` and `#pause` are all in the build.
- **`knowledge/AGENTS.md` points at a codemap that does not exist.** It instructs
  every agent to read `wiki/_codemap.md` and `_codemap.json` *before grepping*.
  Neither file is in the repo; the roadmap separately explains the codemap is
  empty by design for a single-file program. Left for Tucker — it is orientation
  policy, not a fact I should decide.
- **`knowledge/AGENTS.md:179` still has its `{{SEED_NOTE — …}}` placeholder.**

## Open threads
- [ ] Mute verified under Chromium iPhone emulation only. The one claim that
      genuinely needs real WebKit is that pausing `silentEl` actually hands the
      ring/silent switch back — that is the mechanism emulation cannot test.
      Same blocker as [[journal/2026-07-27-mobile-and-ramps]].
- [ ] `knowledge/journal/index.md` says it is generated by `gen_index.py`, which
      is not in this repo (it lives with the `knowledge-layer` skill). The row for
      this entry was added by hand in the generator's exact format.
- [ ] [[journal/2026-07-28-session]] is still `status: in-progress` with only an
      auto-breadcrumb — the *why* behind `9596b47` was never written.
- [ ] Ghost line contrast (0.44 on snow) still needs a human eye. Unchanged.

## Related
- Touched articles: [[wiki/audio]], [[wiki/roadmap]]
