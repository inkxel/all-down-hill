---
type: Journal
date: 2026-07-28
session: codex-pass
description: Codex review of the CPU issue and the new systems; confirmed the title-screen diagnosis and found a weather desync my determinism test was blind to.
status: shipped
related: [[wiki/audio]], [[terrain]], [[plans/2026-07-27-shared-mountains-design]]
---

# Codex pass

## It stalled first, for eleven hours
The run backgrounded via the harness never got past `Reading additional input
from stdin...` — **11h12m elapsed, 0.55s of CPU**. Backgrounding leaves stdin
open and `codex exec` blocks on it forever. Re-running with `< /dev/null`
completed in about six minutes.

**If a codex run produces no output, check elapsed-vs-CPU time before assuming
it is thinking.** Near-zero CPU over a long elapsed means blocked, not busy.

## What it found

**Confirmed, not new:** the title-screen CPU cause. It diffed `HEAD` against its
parent independently and concluded the fix was already in place — useful
corroboration of a diagnosis reached from measurement.

**Applied, and the important one — weather desync.** `startMountain()` rebuilt the
schedule from `runSeed` but left `wSnow / wFog / wWind / wSize / wNext` carrying
over from the previous mountain, then damped toward the new schedule. Two riders
opening the same link could therefore start in **different weather**, depending
on what each had been doing beforehand.

This is precisely the failure mode
[[plans/2026-07-27-shared-mountains-design]] flagged as the nasty one: not
obviously broken, just quietly different.

**My determinism test had the matching hole.** It fingerprinted the weather
*schedule* — which was correctly seed-derived — and never the live scalars. So it
passed, green, while the bug was live. It now covers both.

> A test that checks the derivation but not the state will certify a desync as
> deterministic.

**Applied — untrusted links.** `readLinkFrom` accepted unlimited `r=` params and
`decodeRider` any byte length. Past ~32k samples the `Uint16Array` index buffer
in `buildGhostMesh()` silently overflows. Capped to 6 riders and `GHOST_MAX`
bytes. Verified: a hostile 40-rider fragment carrying a 60KB payload yields 6
riders, longest 440 bytes.

**Applied — `yardSites` unbounded** across re-rides; capped to what the instance
pool can render.

## Not applied
Codex noted `Math.random()` in the title snowfall wrap as a purity issue. It is
title-canvas cosmetics, not terrain, props or weather — outside the determinism
contract, and reseeding it buys nothing. Left alone deliberately.

## Commit log

### 09:10 — 8849920
Document the Codex pass
files: knowledge/decisions/index.md, knowledge/index.md, knowledge/journal/2026-07-27-shared-mountains.md, knowledge/journal/2026-07-28-codex-pass.md, knowledge/journal/2026-07-28-session.md, knowledge/journal/index.md, knowledge/plans/index.md, knowledge/research/index.md, knowledge/wiki/index.md
