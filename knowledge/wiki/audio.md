---
name: audio
description: The WebAudio graph — wind, carve, impacts and the generative music bed, and how they react to the game.
type: subsystem
created: 2026-07-27
last_updated: 2026-07-27
confidence: high
related: [[decisions/2026-07-27-generative-music]], [[physics]]
---

# Audio

Everything is synthesised at runtime. There are no audio files, and there is no
audio library — same constraint as the rest of the game.

## The graph

```
                     ┌─ wind      : looping noise → lowpass  ─┐
                     ├─ carve     : looping noise → bandpass ─┤
one AudioContext ────┤                                        ├─→ master → out
                     ├─ one-shots : whoosh, thud              ─┤
                     └─ music     : pad + notes → reverb      ─┘
```

- **Wind** — filtered noise, gain and cutoff scaled by speed, lifted slightly when airborne.
- **Carve** — bandpassed noise, gain driven by the square of the turn amount so it only speaks on a real carve.
- **Whoosh / thud** — short-lived nodes built per event.
- **Music** — see below.

Continuous parameters are written straight to `.value` each frame and lerped in
JS. **Do not use `setTargetAtTime` per frame** — it queues an event every call
and they pile up.

## Music

Technique from [generative.fm](https://github.com/generativefm), implemented in
raw WebAudio because that project's stack is Tone.js plus sample libraries. See
[[decisions/2026-07-27-generative-music]].

- **Pad** — three detuned voices (2 triangle, 1 sine) holding root/fifth/octave, glided between chords with `setTargetAtTime` on frequency. This is one of the few legitimate uses of it here, because chord changes are ~22s apart.
- **Notes** — minor pentatonic, random octave, soft bell envelope over ~3.4s, straight into the reverb.
- **Reverb** — `ConvolverNode` with a procedurally generated impulse response: noise shaped by an exponential decay. No IR file.

### Reactivity

| game state | effect on the music |
|---|---|
| speed | opens the pad filter, shortens the gap between notes, raises note velocity |
| carving | bends pad cutoff and resonance — the board plays the pad |
| speed (again) | ducks the whole bed, so wind stays on top as it gets loud |
| crash | dims to 25% |
| — | key shifts every ~22s |

Music sits at `MUSIC_GAIN = 0.17`, deliberately under the wind and board.
**There is no mute control** — that is a known gap.

## Removed

The trick chime was cut on 2026-07-27: "didn't add anything and it's too
distracting." Landings still have their thud.

## Context log

### 2026-07-27 — Article created
[[journal/2026-07-27-music-and-dismount]]
