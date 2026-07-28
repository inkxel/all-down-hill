---
type: Plan
date: 2026-07-27
description: Seeded mountains you can share by URL, carrying up to six riders' ghost tracks with no server, plus a top-5 ladder and weather-driven music.
status: approved
deciders: Tucker
related: [[terrain]], [[wiki/audio]], [[decisions/2026-07-26-single-file-cdn-three]]
---

# Shared mountains — design

## What it is

A run happens on a *named* mountain. You can share that mountain by link. Whoever
opens it rides the same terrain, in the same weather, and sees the faint tracks
of everyone who rode it before — each line ending exactly where that rider went
down.

They can share it onward. Their link carries their run plus the ones already
there. The mountain accumulates.

## Why it earns its keep

The terrain is currently generated and then indifferent to the player. This makes
the mountain a *place*: it has an identity, it can be returned to, and it carries
evidence of what people did on it. That is the difference between "procedural"
and "reactive", and it is the only version of the ideas we discussed where the
world changes because of something a person did.

It also produces the honest hook the README currently lacks: **ride someone
else's mountain and see where they fell.**

## The constraint that shapes everything

**No server.** The game is a static file on GitHub Pages and stays that way
([[decisions/2026-07-26-single-file-cdn-three]]).

So ghosts travel *in the URL fragment*. The fragment never reaches a server, has
no practical length limit for our sizes, and survives the repo being untouched
for years. Sharing chains because each link carries what it received plus the new
run.

The cap on riders is **social, not technical** — browsers would take fifty, but
nobody pastes a 20KB URL. Six keeps a link under ~2KB and pasteable.

---

## 1. Seeded mountains

A run is defined by a 32-bit `runSeed`. Everything a rider can see must derive
from it, or two people on the same link get different mountains.

Three places currently break that:

| what | now | change |
|---|---|---|
| terrain noise | `seedNoise(20260726)` as a load-time IIFE | callable; re-seed from `runSeed` on new run |
| prop scatter | `mulberry32((k * 9781 + 13) \| 0)` | mix in `runSeed` |
| weather | two bare `Math.random()` calls | seeded schedule precomputed from `runSeed` |

Cosmetic seeds stay fixed and must **not** be touched — title snowfall, stars,
rock vertex jitter, weather particles, speed lines. They are not terrain and
re-seeding them buys nothing.

Re-seeding the noise rebuilds a 512-entry permutation table; trivial. All chunks
must be rebuilt on a new run, which already happens.

**Weather determinism.** Precompute a schedule at run start from a seeded PRNG:
a list of `{ at, state }` covering a long run. `updateWeather` reads the schedule
by elapsed time instead of rolling dice. Two riders on the same link then hear
the same storm arrive at the same distance.

**Mountain code.** `runSeed` shown as base36, e.g. `7K2QX`. That is the
mountain's name — on the title screen when arriving via a link, and on the
leaderboard.

## 2. Recording a ghost

Sample every `GHOST_STEP` (8m) of distance travelled:

- lateral `x`, quantised to 7 bits over the ±70 playfield (~1.1m resolution)
- 1 bit: airborne

One byte per sample. Written into a preallocated `Uint8Array` — **no per-frame
allocation**, consistent with the rest of the hot loop. 600 samples covers ~4.8km.

Recording stops where the run ends. The line's end *is* the crash — no separate
marker data needed.

Quantisation stepping is smoothed at render time by interpolating the polyline;
1.1m is invisible once the ribbon is built.

## 3. URL format

```
#m=<seed base36>&r=<rider>&r=<rider>…
```

Each rider: `<score base36>.<base64url of the sample bytes>`

~250 bytes per 1,700m run → ~330 chars base64. Six riders ≈ 2KB.

On load: parse the fragment, adopt the seed, decode ghosts. No fragment means a
fresh random seed and no ghosts.

## 4. Rendering ghosts

Build one ribbon mesh per ghost at load, in **absolute world coordinates**. Float32
error at 30km is ~0.004 units — invisible on a decorative line, so no rebasing
needed here (unlike the live carve trail, which sits under the camera).

- vertices at `(x_i, terrainHeight(x_i, z_i) + 0.06, z_i)`
- reuse the carve-trail shader; lower opacity, slightly cooler tint
- airborne samples get alpha 0, so a ghost's jumps read as gaps in the line
- the line simply stops where they crashed

~1,000 vertices per ghost, 6 ghosts. Negligible.

## 5. The ladder

On run end:

1. `survivors` = top 5 of (existing ghosts ∪ your run) by score
2. `shareList` = `survivors`, plus your run if it didn't make the cut (max 6)

So **anyone can always share**, and a weak run propagates exactly one hop — your
friend sees your line, then it falls out. Good runs persist and entry gets harder
over time.

Rejected: gating sharing on placing top-3. It reads better as prestige but
sharing *is* the distribution mechanism — gate it and a strong mountain locks,
and the person you sent it to can't pass it on. That kills the behaviour the
feature exists to create.

The existing `localStorage` leaderboard is **separate** and stays: it is your
personal best across all mountains. The ladder is per-link.

## 6. Sharing

On the leaderboard screen: **Share this mountain** → build the URL, write it with
`navigator.clipboard.writeText`, confirm inline. Fall back to showing the URL in
a selectable field where the clipboard API is unavailable.

Title screen, when arrived via a link: show the mountain code and how many riders
have been there.

## 7. Weather → music

Fold `wSnow` / `wFog` into `updateMusic`:

- heavy snow closes the pad filter and thins note density — the mountain gets quieter as it closes in
- clear weather opens it back up

Small, and it completes the loop: same seed means the same storm, so two people
riding the same link hear the same thing happen.

---

## Risks

**Determinism is the whole feature.** Any `Math.random()` leaking into terrain,
props or weather silently desynchronises two riders on the same link, and it will
not be obvious — the mountains will look similar. This needs a real test: load the
same seed twice, sample terrain height and the weather schedule at fixed points,
assert identical.

**Ghost fidelity vs URL size** is the tuning knob. 8m/7-bit is the starting point.

**A ghost is not a replay.** It has no timing, so it cannot be raced against — it
is a trace, not a rival. That is deliberate; timing data would roughly double the
payload for something the design does not use.

## Build order

1. Seeded mountains + determinism test — nothing else works without it
2. Ghost recording and encode/decode round-trip
3. Ghost rendering
4. Ladder + share UI
5. Weather → music

Each stage is independently shippable; 1–3 are worth landing before the social
layer exists.
