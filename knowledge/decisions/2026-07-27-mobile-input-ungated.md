---
type: Decision
date: 2026-07-27
description: Touch listeners attach unconditionally, the iOS motion permission is requested from touchend, and a tap always jumps.
status: accepted
supersedes: the zone-split touch fallback in [[decisions/2026-07-27-altos-jump-model]]
deciders: Tucker
related: [[physics]], [[wiki/mobile]]
---

# Decision: Ungate touch input; request motion permission from touchend

## Context
Mobile was completely non-functional on Tucker's iPhone across Firefox, Chrome
and Safari — all three are WebKit on iOS, so this was one bug, not three. Neither
tap-to-jump nor tilt steering did anything.

Two compounding causes:

1. `setupMotion()` ran from `pointerdown`. iOS does not reliably treat that as the user gesture required by `DeviceOrientationEvent.requestPermission`, so the request was rejected and tilt never armed.
2. That rejection set `touchSteer = true`, and the fallback split the screen into zones where the lower 58% steered. A natural thumb tap therefore steered instead of jumping — the jump looked entirely dead.

## Decision
- Request motion permission from a **`touchend`** listener. iOS accepts it, and unlike `click` it still fires after the touch sequence is `preventDefault`ed.
- **A touch is a jump until it is dragged** past 16px, at which point it converts to a steer. No zones.
- **Attach touch listeners unconditionally.** They were gated on an `isTouch` heuristic; one wrong guess left the game with no input at all.
- `steerInput()` reads keyboard, then tilt, then touch, rather than branching on device detection.

## Consequences
- **Positive:** a tap always jumps, in either steering mode. That was the actual complaint.
- **Positive:** a wrong device guess can no longer disable input — detection now only affects presentation (prompt wording, particle counts, DPR cap).
- **Negative:** in the touch-steer fallback, starting a carve also fires a jump, because the jump registers before the drag threshold is crossed. Acceptable in a degraded mode; invisible when tilt is granted.
- **Neutral:** `isTouch` is retained, but nothing load-bearing depends on it.

## Dissent / Alternatives Considered
- **Keep zones but shrink the steer strip** — still fails the "tap anywhere to jump" expectation, and any zone boundary is invisible to the player.
- **Request permission from `click`** — `click` never fires, because the touch handlers call `preventDefault()`, which suppresses the synthetic mouse events.
- **Fix the `isTouch` heuristic instead of ungating** — treats a symptom. The heuristic was in fact correct on iOS; the failure was downstream. Ungating removes the whole class of bug.

## Sources
- [[journal/2026-07-27-mobile-and-ramps]]
