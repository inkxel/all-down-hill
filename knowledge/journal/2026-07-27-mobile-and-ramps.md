---
type: Journal
date: 2026-07-27
session: mobile-and-ramps
description: Disabled ramp placement without deleting the machinery; found and fixed three separate reasons mobile was non-functional on iPhone.
status: shipped
related: [[mobile]], [[decisions/2026-07-27-mobile-input-ungated]], [[decisions/2026-07-27-ramps-removed-not-deleted]]
---

# Mobile rescue + ramps switched off

## Context
Tucker after playing: landings feel much better, but pull the ramps entirely —
they don't play or look right — while keeping the physics and barrel roll for a
future redesign. And mobile "doesn't work at all," across Firefox, Chrome and
Safari on iPhone: no touch jump, no tilt steering, browser chrome covering the
screen, title cropped in both orientations, rider invisible in portrait.

## Ramps
`RAMPS_ENABLED = false` gates placement only. `rampRise`, the edge taper, spin
arming, barrel-roll rotation and `settleSpin` all stay.
[[decisions/2026-07-27-ramps-removed-not-deleted]]

Consequence worth remembering: **barrel rolls are now unreachable in play**, since
`spinAxis` is only ever set by an angled kicker. The path is live but dead.

## Mobile — three bugs, not one
All three iOS browsers are WebKit, so this was never a per-browser issue.

**1. The permission gesture.** `setupMotion()` ran from `pointerdown`, which iOS
doesn't reliably accept for `requestPermission`. The request was rejected, tilt
never armed, and the game silently fell back to touch steering. Moved to
`touchend` — which iOS accepts and which, unlike `click`, still fires after the
touch handlers `preventDefault()`.

**2. The fallback ate the jump.** In touch-steer mode the screen was split into
zones and the lower 58% steered. A natural thumb tap therefore steered instead of
jumping, so the jump looked completely dead. Replaced with: a touch is a jump
until dragged past 16px. Both symptoms Tucker reported traced to these two
compounding causes.

**3. The viewport.** iOS's layout viewport extends behind the browser chrome, so
sizing to `innerHeight` put the bottom of the scene — and the rider in portrait —
underneath the nav bar. Everything now derives from `visualViewport` via
`--app-w`/`--app-h` and a shared `.layer` class.

Also: wordmark resized off the short side so it fits both orientations, and the
portrait camera pitches further down.

## An error I made mid-fix, caught by testing
While rewriting the touch section I **deleted the `addEventListener` block along
with it**, so no touch listener was registered at all. The probe caught it
immediately (`sawTouch: false`, `activeTouches.size: 0`) — a visual check would
not have, because the title screen and rendering looked perfectly fine.

Worth noting the pattern: a large block replacement silently dropped a few lines
at the boundary. Asserting on behaviour rather than eyeballing the diff is what
found it.

## What I could not verify — and gave up on
These fixes are verified against **Chromium with iPhone emulation** plus a
scripted emulation of iOS's `requestPermission` activation semantics. **Not**
against real WebKit.

I tried three times to install Playwright's WebKit to close that gap. It
downloads to 100% and then never extracts — the first attempt hit the 10-minute
tool timeout while holding `~/Library/Caches/ms-playwright/__dirlock`, which made
the next two fail on the stale lock and then stall identically after a clean
reset. Abandoned rather than burn more time; the partial install and lock were
removed so the Chromium-based suite is unaffected.

What that leaves genuinely unverified on the real engine:
- whether `touchend` grants transient activation for `requestPermission` in Safari (high confidence — it is the documented and widely-used gesture, but unproven here)
- `visualViewport` behaviour as iOS chrome slides in and out

Everything else — the tap-vs-drag model, listener attachment, tilt mapping,
layout sizing — is engine-independent and is verified.

## Open threads
- [ ] Tucker to re-test on iPhone across the three browsers.
- [ ] Barrel roll unreachable until ramps (or another angled launch) return.
- [ ] Re-run the mobile suite under Playwright WebKit. Install stalls after download on this machine (three attempts); needs a different approach or a manual browser install.

## Commit log

### 15:28 — 33eb6e0
Document the mobile rescue and the ramp switch-off
files: knowledge/decisions/2026-07-27-mobile-input-ungated.md, knowledge/decisions/2026-07-27-ramps-removed-not-deleted.md, knowledge/decisions/index.md, knowledge/journal/2026-07-27-landing-fix.md, knowledge/journal/2026-07-27-mobile-and-ramps.md, knowledge/journal/index.md, knowledge/wiki/index.md, knowledge/wiki/mobile.md, knowledge/wiki/roadmap.md
