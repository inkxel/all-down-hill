---
type: Journal
date: 2026-07-27
session: music-and-dismount
description: Made both grind dismounts safe with the jump paying more, removed the trick chime, and added a reactive generative music bed.
status: shipped
related: [[wiki/audio]], [[decisions/2026-07-27-generative-music]]
---

# Music + grind dismount

## Context
Tucker: mounting a log late left no room to jump off before the end, making the
grind an unavoidable crash — so make rolling off safe and the jump merely *worth
more*. Also cut the trick chime. And: is there a world where we add calm
generative music, kept low, reactive to speed and to the game's own sounds? With
prior-art research rather than building blind.

## Grind dismount
Rolling off the end now drops you back to the snow instead of crashing. Popping
off is the skilled exit: double chain plus a short boost.

| dismount | survived | avg chain |
|---|---|---|
| ride off the end | 6/6 (previously always fatal) | 1.8 |
| jump off | 5/5 | 2.6 |

## Music — what the research actually concluded
[generative.fm](https://github.com/generativefm/generative.fm) is *the* browser
generative-music project. It is also **not usable here**: Tone.js plus recorded
sample libraries, against a project whose two hardest constraints are one
self-contained file and zero assets.

So the same call as the tilt work — take the technique, leave the dependency.
Small fixed scale, notes at randomised intervals, long envelopes, reverb over
everything. Written in raw WebAudio on the graph the wind and carve already
occupy, which is what makes it genuinely reactive rather than a parallel system
guessing at game state. [[decisions/2026-07-27-generative-music]]

Measured over a speed ramp: pad cutoff 479 → 656 Hz, note interval shortening,
and duck falling to 0.41 during a crash.

## Second pass — mix and legibility
Tucker: the music "gets really quiet and tough to hear," and "I can't tell if my
riding effects the music at all." The synth character itself was right.

Three separate causes, only one of which was volume:
1. Level too low (0.17).
2. The speed duck cut 28% — biting hardest exactly where the player spends their time, so it faded out precisely when they were paying attention.
3. **Reactivity was real but unreadable.** Note density was the only tempo cue. It measurably changed and was still impossible to perceive.

Fix for (3) is the interesting one: the pad now breathes through an LFO whose
rate tracks speed. A tempo change is unmistakable in a way a density change is
not. Also pushed the carve response far harder — and finally measured it while
actually riding, which closes the gap flagged in the previous entry:

| | cutoff | Q |
|---|---|---|
| straight | 496 Hz | 3.0 |
| carving | 2466 Hz | 9.5 |

The opening swoop Tucker liked was an accident (oscillators defaulting to 440Hz
and gliding to the first chord). It is now deliberate and commented so a future
pass doesn't tidy it away.

## Pause
ENTER / ESC on desktop, two fingers on touch. Sim, clock and day cycle all
freeze. Resuming re-learns the tilt neutral — the phone has almost certainly
moved — and grants a short grace. Verified a single finger still jumps rather
than pausing.

## Third pass — no audio on mobile at all
Tucker: no audio on mobile, and he had assumed his volume was down.

**Web Audio is silenced by the iOS ring/silent switch; HTML5 media elements are
not.** Nothing to do with volume, gestures, or the AudioContext — and completely
invisible to every test I can run, because neither desktop Chromium nor its
iPhone emulation has a silent switch.

Fixed with the AudioSession API plus the established looping-silent-element
trick. Verified the mechanism (session set to `playback`, element in the DOM,
looping, playsinline, Blob src, context `running`, no duplicate on a second
gesture) — but **not the actual mute-switch behaviour**, which needs his phone.

Note this makes the game override the silent switch. Standard for games,
deliberate, and reversible.

## Open threads
- [ ] **No mute control** — now more pressing, since the game overrides the iOS silent switch. A player who mutes their phone has no way to mute the game.
- [ ] iOS silent-switch bypass verified by mechanism only; real-device confirmation outstanding.
- [ ] Tucker to judge the mix. `MUSIC_GAIN` is 0.17.
- [x] ~~Carve → filter link unverified~~ — measured 2026-07-27: 496Hz/Q3 → 2466Hz/Q9.5 while riding.
- [ ] Grind capture may now be too sticky after the previous pass widened it.

## Commit log

### 19:08 — 840e27d
Document the music system and grind dismount
files: knowledge/decisions/2026-07-27-generative-music.md, knowledge/decisions/index.md, knowledge/journal/2026-07-27-music-and-dismount.md, knowledge/journal/2026-07-27-tilt-and-hop.md, knowledge/journal/index.md, knowledge/wiki/audio.md, knowledge/wiki/index.md

### 19:30 — fb7388e
Louder, audibly reactive music; pause on Enter / two-finger tap
files: index.html

### 19:32 — 374de8b
Document the music mix pass and pause
files: knowledge/journal/2026-07-27-music-and-dismount.md, knowledge/wiki/audio.md

### 19:51 — 0418b14
Play audio on iOS even with the ring/silent switch on
files: index.html

### 19:52 — e64b262
Document the iOS silent-switch audio trap
files: knowledge/journal/2026-07-27-music-and-dismount.md, knowledge/wiki/audio.md

### 20:03 — 3c323a1
Rewrite README with voice
files: README.md

### 20:07 — 9a960b7
Demote the "nothing was drawn" hook
files: README.md
