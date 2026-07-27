---
type: Research
date: 2026-07-26
description: Full ledger of the Codex correctness pass and the ponytail complexity pass — every finding, what was applied, and the reasoning behind each decline.
related: [[journal/2026-07-26-build-and-review]]
---

# Review passes — findings ledger

Two scoped reviews were run on the completed build. Recording the *declines* is
the point of this document: without it, the same findings get re-raised and
re-argued next time.

## Pass 1 — Codex, correctness

Scope given: runtime errors, physics/math bugs, per-frame allocation, chunk
recycling leaks, mobile input edge cases, GitHub Pages breakage. Explicitly
out of scope: refactors, file splitting, build tooling, removing effects.

### Applied (6)

| Severity | Finding | Fix |
|---|---|---|
| High | Obstacle collision was a point test — tunnels at 42 u/s when dt hits the 50 ms clamp | Swept segment test (`sweptDistSq`) |
| Medium | 4 s DeviceOrientation permission timeout fell back to touch permanently, even if the user granted later | Removed the timeout; touch steers while the iOS sheet is open, tilt takes over on grant |
| Medium | `visibilitychange` / `pagehide` didn't clear held touches | Shared `clearInput()` |
| Medium | Orientation change kept the stale tilt baseline | Re-baseline on `orientationchange` and on resume |
| Low | Title canvas gradients allocated every frame | Built in `sizeTitle()` |
| Low | WebGL context failure killed the page | `try/catch` leaves the dawn title up with a message |

Also added independently: a 0.9 s respawn grace, after measuring repeat crashes
at 9–11 m — you were respawning inside the tree you just hit.

### Declined (2)

**"Trail gaps at low FPS."** Misdiagnosed. The tail vertex pair is rewritten every
frame, so the ribbon never disconnects; a dropped step only lowers sample density.
The proposed fix (consume multiple steps per frame) would duplicate the current
position N times and *shorten* the trail — strictly worse.

**"Skip rendering the 3D scene behind the title screen."** True that it costs
frames, but that live scene is what makes "input cross-fades into the 3D game" a
cross-fade rather than a cut. Deliberate.

### Confirmed clean
No syntax/TDZ/hoisting issues. No chunk-recycling leak, no InstancedMesh slot
corruption, no particle pool leak. No Pages subpath or CDN issue.

## Pass 2 — ponytail-review, complexity

Treated as advisory. Tucker's standing instruction: apply findings that remove
genuine redundancy, but **do not** apply anything that removes or simplifies
particles, carve trails, screen shake, slow-mo, speed lines, the day/night cycle,
weather states, or audio — those are deliberate.

### Applied — net −16 lines
- `clamp` / `lerp` / `smoothstep` / an exponential-approach helper were all hand-rolled; `THREE.MathUtils` ships all four (`damp` is character-for-character identical to the helper).
- Two identical float32 re-base blocks → one `rebase()`.
- Speed ratio and visible-snowflake count were each derived in more than one per-frame function → computed once.
- An unused scratch `Vector3`, an unused tilt variable, a `box()` rotation parameter never passed non-zero.

**This pass had a cost.** `MathUtils.smoothstep` is `(x, min, max)`, not the
`(edge0, edge1, x)` convention of the replaced helper. The next session wrote an
inverted smoothstep out of habit and silently disabled every ramp in the game.
See [[journal/2026-07-27-feel-rework]]. Swapping a utility for a stdlib
equivalent is not free when the argument order differs.

### Declined (2)

**`mergeGeos` → `BufferGeometryUtils.mergeGeometries`.** That is a separate addon
module — a second network fetch and failure point — to remove 18 lines that run
once at startup. Violates [[decisions/2026-07-26-single-file-cdn-three]].

**`mulberry32` → `MathUtils.seededRandom`.** `seededRandom` advances a single
module-global seed, so interleaved per-chunk generators would corrupt each
other's determinism. Not a drop-in.

## Takeaway

The correctness pass earned its keep (the tunnelling bug was real and
unfindable by eye). The complexity pass was net positive but introduced a
regression, and neither pass found the three bugs that actually made the game
look wrong — those came from screenshotting the running game.
