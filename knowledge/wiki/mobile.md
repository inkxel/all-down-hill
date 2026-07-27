---
name: mobile
description: How touch, tilt and the viewport work on phones — and the traps that made mobile completely non-functional once.
type: subsystem
created: 2026-07-27
last_updated: 2026-07-27
confidence: high
related: [[physics]], [[decisions/2026-07-27-mobile-input-ungated]]
---

# Mobile — input and viewport

## All three iOS browsers are the same engine

Firefox and Chrome on iPhone are both WebKit — Apple requires it. A bug
reproducing in "all three browsers" on iOS is one bug, and Playwright's WebKit
(or Chromium's iPhone emulation for layout) reproduces it locally.

## Input

Touch listeners attach **unconditionally**. They used to be gated on an
`isTouch` heuristic, which meant a single wrong guess left the game with no
input at all. `isTouch` now only affects presentation: prompt wording, particle
counts, DPR cap.

`steerInput()` reads keyboard → tilt → touch in priority order rather than
branching on device type.

### A tap is a jump

A touch counts as a jump until it is dragged past 16px, at which point it
converts to a steering touch and stops contributing to jump-hold.

**Do not reintroduce screen zones.** The previous fallback steered on any touch
in the lower 58%, so a natural thumb tap steered instead of jumping and the game
felt completely unresponsive. See
[[decisions/2026-07-27-mobile-input-ungated]].

### The iOS motion permission gesture

`DeviceOrientationEvent.requestPermission()` needs transient user activation.

- **`pointerdown` is not reliably accepted.** This was the original bug.
- **`click` never fires** here, because the touch handlers call `preventDefault()`, which suppresses synthetic mouse events.
- **`touchend` works** and is what the game uses.

There is no arbitrary timeout on the permission promise — the iOS sheet can sit
open for a while, so the game steers by touch in the meantime and upgrades to
tilt whenever the promise resolves.

Tilt reads `gamma`, remapped per `screen.orientation.angle` so landscape works,
against a baseline captured on start and recaptured on orientation change and on
resume from background.

## Viewport

**iOS's layout viewport extends behind the browser chrome.** `window.innerHeight`
therefore overstates what you can see, and sizing the canvas to it hides the
bottom of the scene — which is where the rider sits in portrait.

Everything is driven off `window.visualViewport` instead:
- `--app-w` / `--app-h` CSS custom properties, consumed by the `.layer` class that every full-screen fixed element uses.
- `renderer.setSize()` and `camera.aspect` use the same numbers.
- Re-run on `resize`, `visualViewport.resize`, `visualViewport.scroll` (chrome sliding in and out), and twice after `orientationchange` because iOS reports stale sizes immediately after the flip.

### You cannot hide the browser chrome from a page

No API does this on iOS. What the game does instead is fit the visible area
exactly so nothing important is ever behind the bar. For true fullscreen the page
sets `apple-mobile-web-app-capable`, so **Add to Home Screen** launches it
without chrome.

## Both orientations

The camera derives a `tall` factor from aspect ratio and pitches further down and
sits higher as the frame gets taller, because vertical FOV means a portrait
viewport otherwise shows mostly sky. The wordmark is sized off the short side so
it fits either way round.

## Context log

### 2026-07-27 — Article created
Written after mobile was found to be entirely non-functional on iPhone.
[[journal/2026-07-27-mobile-and-ramps]]
