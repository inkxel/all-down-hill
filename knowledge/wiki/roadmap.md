---
title: Roadmap
type: Reference
---

# Roadmap

The "not now, but don't forget" pile.

## Awaiting Tucker's feel notes
- **Landing feel after the settle assist.** Crash rate under an adversarial bot is 32% (was 100%); no idea what a human rate looks like.
- **Rider pose constants are provisional.** Shipped Ink to get it in his hands; crouch depth, arm swing and scarf sweep are all first-pass numbers.
- **`RIDER_SCALE = 1.15`** is a readability compromise, not a considered proportion. Real scale is 1.0.

## Known-soft
- **Crown bands may be too generous.** Rock and tree crowns are deliberately forgiving; unplayed by a human yet.
- **`chain` feeds `mult`, capped at 9**, so long chains saturate quickly. May need its own scale.
- **Section rhythm is fixed-period (350 m).** A long session will feel the loop. Density ramping masks it; it doesn't solve it.
- ~~**Barrel roll is hard to land.**~~ **Resolved 2026-07-27** — was systemic, not roll-specific: any hold past ~90 ms crashed on either axis. Fixed with a settle-to-nearest-turn assist. [[decisions/2026-07-27-spin-settle-assist]]
- ~~Small kickers easy to miss~~ — moot while ramps are disabled.

## Deferred features
- **Ramps are switched off**, machinery intact behind `RAMPS_ENABLED`. Tucker wants to redesign them before they return. ~~Until then **barrel rolls are unreachable** — `spinAxis` is only set by an angled kicker.~~ **Corrected 2026-07-31** — rolls have been reachable since the feel pass: `spinAxis` is set from carve speed at `index.html:2902` / `:2920` when lateral speed exceeds `STEER_MAX * CARVE_ROLL`. Carving hard into a takeoff *is* the launch, and it replaced the ramp dependency rather than waiting on it. [[journal/2026-07-27-shared-mountains]]
- **Verify mobile under real WebKit.** The 2026-07-27 fixes are verified against Chromium iPhone emulation + scripted iOS permission semantics only. Playwright's WebKit install stalls after download on this machine (three attempts, incl. a stale-lockfile cascade). Needs a different install route before this can be closed.
- **Rider treatments 2–5** are specified and rendered but unbuilt. Halo needs its vertex-colour rim bake restored; the rim is baked against a fixed light direction so it won't follow the day cycle without a shader.
- **Leaderboard is `localStorage` only.** No accounts, no server, no sharing beyond the JPEG card. *(Partly overtaken — the per-link ladder in [[plans/2026-07-27-shared-mountains-design]] does travel between people. The `localStorage` board is still the separate personal one.)*
- ~~**No pause.** Deliberate per the original spec, but the continue panel now proves a pause surface would fit.~~ **Corrected 2026-07-31** — pause shipped: `pauseGame()` / `resumeGame()`, the `#pause` veil, `ENTER`/`ESC` on desktop and two fingers on touch.
- **No mute control** — tracked in journal open threads rather than here, twice. **Closed 2026-07-31**: `M` or the bottom-right toggle, held on `master` and persisted to `adh_mute`. [[wiki/audio]]

## Technical debt
- **The generated `_codemap.md` is empty** because the program is one HTML file and `codemap.py` doesn't descend into inline `<script>`. [[index-map]] is the hand-maintained substitute and will drift.
  - **2026-07-31** — it is not empty, it is *absent*: neither `wiki/_codemap.md` nor `_codemap.json` is in the repo. `knowledge/AGENTS.md` still instructs every agent to read them **before grepping**, which sends each new session to a missing file. Needs Tucker's call — either generate them, or cut that paragraph and point at [[index-map]] instead.
- **`knowledge/AGENTS.md:179` still carries its `{{SEED_NOTE — …}}` template placeholder** from the scaffold.
- **No unit-test surface.** All verification is Playwright against a running browser. Fine at this size; would not scale.
- **Three.js is pinned to `0.185.1`.** No lockfile, so an upgrade is a manual edit and a full manual re-verify.
