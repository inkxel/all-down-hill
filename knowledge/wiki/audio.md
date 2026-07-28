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

## iOS silences Web Audio with the ring switch

**Web Audio is muted by the iOS silent switch. HTML5 media elements are not.**
This is why the game was silent on a phone with the switch flipped while desktop
was fine — and it is invisible in any desktop or emulator test.

Two fixes applied together in `unmuteIOS()`:

1. `navigator.audioSession.type = 'playback'` — Safari 16.4+, the standards route.
2. A **looping silent clip in a media element**, which promotes the audio session out of the `ambient` category. Technique from [swevans/unmute](https://github.com/swevans/unmute) and [feross/unmute-ios-audio](https://github.com/feross/unmute-ios-audio), reimplemented rather than imported — the silent WAV is generated at runtime into a Blob, so it stays an "everything procedural" build.

Two things that will break it if changed:
- **The element must be appended to the document.** A detached media element does not reliably promote the session.
- **It retries on every touch** until the context reaches `running`; the first gesture can land before iOS is willing.

**Consequence worth knowing:** the game now plays audio even when the phone is
switched to silent. That is normal for games but it is a deliberate choice, not
an accident. Removing `unmuteIOS()` restores respect for the switch.

## Music

Technique from [generative.fm](https://github.com/generativefm), implemented in
raw WebAudio because that project's stack is Tone.js plus sample libraries. See
[[decisions/2026-07-27-generative-music]].

- **Pad** — three detuned voices (2 triangle, 1 sine) holding root/fifth/octave, glided between chords with `setTargetAtTime` on frequency. This is one of the few legitimate uses of it here, because chord changes are ~22s apart.
- **Notes** — minor pentatonic, random octave, soft bell envelope over ~3.4s, straight into the reverb.
- **Reverb** — `ConvolverNode` with a procedurally generated impulse response: noise shaped by an exponential decay. No IR file.

### Reactivity

| game state | effect on the music | measured |
|---|---|---|
| speed | pad LFO rate — the bed breathes faster | 0.55 → 2.05 Hz |
| speed | opens the pad filter | 340 → ~1840 Hz |
| speed | shortens the gap between notes | ~3.2s → ~1.1s |
| carving | bends pad cutoff and resonance | 496 Hz / Q 3 → 2466 Hz / Q 9.5 |
| speed | light duck so wind stays on top | −10% at full speed |
| crash | dims | 35% |
| — | key shifts every ~22s | |

**The LFO is the one that makes reactivity legible.** Note density alone was the
original cue and it was unreadable — Tucker played it and could not tell the
music responded at all. A tempo change is unmistakable where a density change is
not.

The descending swoop on the first note is **deliberate**: the pad oscillators
start high and glide into the first chord. Don't "fix" it.

Music sits at `MUSIC_GAIN = 0.30`. It was 0.17 with a 28% speed duck, which made
it vanish exactly where the player spends most of their time.
**There is no mute control** — that is a known gap.

## Removed

The trick chime was cut on 2026-07-27: "didn't add anything and it's too
distracting." Landings still have their thud.

## Context log

### 2026-07-27 — Article created
[[journal/2026-07-27-music-and-dismount]]
