---
name: atmosphere
description: Sky, day/night cycle, weather states, fog and the horizon parallax — how the scene's mood is driven from two numbers.
type: subsystem
created: 2026-07-27
last_updated: 2026-07-27
confidence: high
related: [[terrain]], [[decisions/2026-07-26-light-intensity-pi-scale]]
---

# Atmosphere — sky, light, weather

## Day/night

`DAY` is an eight-keyframe table (dawn → sunrise → day → golden → dusk → night →
pre-dawn → dawn) over a 240-second cycle. `applyDay(dayT)` finds the bracketing
pair and lerps *everything* from it: sky gradient top/bottom, horizon glow colour
and intensity, sun colour and intensity, hemisphere sky/ground, ambient, star
opacity, sun elevation, snow tint and mountain tint.

Adding a new mood means adding a row, not writing code.

### Two traps baked into the table

**Light intensities are ~π× "intuitive" values.** three.js r155 dropped the legacy
π scaling on ambient/hemisphere/directional lights, so an intensity that looks
right by old intuition renders about three times too dark. See
[[decisions/2026-07-26-light-intensity-pi-scale]].

**The sun's azimuth arc is deliberately narrow** (`-1.05 + dayT * 1.2` radians).
A full sunrise-to-sunset sweep is more realistic but puts the sun *behind* the
fall line for much of the cycle, which kills `dot(N, L)` on a slope whose normal
tilts downhill — the snow goes flat and muddy. The narrow arc keeps the sun
ahead-of-slope all cycle and keeps the moon on screen at night.

## Backdrop layers

Sky dome, stars, sun/moon sprites and five horizon mountain layers all follow the
camera each frame. The mountain layers apply a per-layer screen-parallax factor to
camera x and y, so carving and descending both produce depth cues. Mountains use
`fog: false` so they stay crisp above the fogged terrain horizon.

## Weather

Four states — clear, light, heavy, windy — cycling on their own 22–42 s timer,
independent of the day phase. Each carries snow density, a fog-distance multiplier,
wind drift and flake size; all four are damped toward the target so transitions
cross-fade. Density is applied via `setDrawRange` on the snow `Points`, so unused
flakes cost nothing.

Fog colour tracks the sky's bottom colour every frame, which is what makes distant
terrain dissolve into the horizon instead of ending at a visible edge.

## Context log

### 2026-07-27 — Article created
[[journal/2026-07-26-build-and-review]]
