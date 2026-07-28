---
type: Decision
date: 2026-07-27
description: A low generative music bed built in raw WebAudio using generative.fm's technique, sharing the graph with wind and carve so the game plays the music.
status: accepted
deciders: Tucker
related: [[wiki/audio]], [[decisions/2026-07-26-single-file-cdn-three]]
---

# Decision: Generative music in raw WebAudio, technique borrowed not code

## Context
Tucker asked for "calm mellow generative synth" music, kept low because he likes
hearing the wind and the board, and ideally with the game's own sounds feeding
the music — "music speeds up slightly as you speed up." He asked for prior art
research rather than building blind.

[generative.fm](https://github.com/generativefm/generative.fm) is the reference
implementation for browser generative music. It is not directly usable here: it
depends on **Tone.js** and on **recorded sample libraries**, which break both
this project's hardest constraints — one self-contained file, and zero assets
([[decisions/2026-07-26-single-file-cdn-three]]).

## Decision
Take the technique, not the dependency — the same call made for tilt in
[[decisions/2026-07-27-tilt-gravity-projection]].

From generative.fm's approach: a small fixed scale, notes scheduled at
randomised intervals, long soft envelopes, everything blurred through reverb.
Implemented in raw WebAudio on the graph the wind and carve already use, which
is what makes the reactivity possible at all:

- minor pentatonic over a three-voice detuned pad; key shifts every ~22s
- reverb from a procedurally generated impulse response (noise with an exponential tail) — no IR file
- **speed** opens the pad filter and shortens the gap between notes
- **carving** bends filter cutoff and resonance, so the board is playing the pad
- music **ducks under the wind** as speed rises, and dims on a crash

Sits at 0.17 gain, deliberately beneath the wind and board.

## Consequences
- **Positive:** no new dependency and no assets; the constraint holds.
- **Positive:** because it shares the audio graph, reactivity is direct rather than a parallel system guessing at game state.
- **Negative:** a hand-rolled generative system is far less musically sophisticated than generative.fm's sample-based pieces. Oscillators, not instruments.
- **Negative:** no mute control. If the music is unwanted there is currently no way to turn it off without an edit.
- **Neutral:** a handful of always-on oscillators plus one short-lived node per note — negligible cost.

## Dissent / Alternatives Considered
- **Import Tone.js from a CDN** — would make the music far easier to write and much richer. Rejected: a second runtime dependency, and Tone.js is ~200KB before any sound is made. Worth revisiting only if the music becomes a headline feature.
- **Ship an audio file** — flatly incompatible with "everything procedural", and would be the first asset in the repo.
- **Analyse the wind bus with an AnalyserNode to drive the music** — literally "the sounds affect the music", but circular: wind level is already a function of speed, so it would be an indirect path to the same number.

## Sources
- [[journal/2026-07-27-music-and-dismount]]
