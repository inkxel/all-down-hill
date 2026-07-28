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

## Open threads
- [ ] **No mute control.** If the music is unwanted there is no way to turn it off without editing the file.
- [ ] Tucker to judge the mix. `MUSIC_GAIN` is 0.17.
- [ ] The carve → filter link is verified in code but was not observed under the bot, which does not carve while sampling. Needs a human ear.
- [ ] Grind capture may now be too sticky after the previous pass widened it.

## Commit log
