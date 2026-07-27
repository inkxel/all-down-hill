---
type: Research
date: 2026-07-27
description: Five graphic treatments for the rider rendered on identical geometry, and why Ink was chosen.
related: [[rider]], [[journal/2026-07-27-feel-rework]]
---

# Rider comps — five graphic treatments

Tucker's note on the original box rider: "our character looks really bad." When
offered three directions he asked to **combine** rounded low-poly with graphic
silhouette, and to see five comps before committing.

## Method

All five share **identical geometry and pose** so the comparison is purely
graphic. Rendered in the game's actual bright-day lighting rig at two distances:
a hero angle and roughly gameplay read.

Building them required getting the rig right first, which took several passes.
Recorded because the failure modes recur:

- Ground displacement wasn't zeroed at the origin, so the snow plane sat 0.5 m above the board and buried the rider to the knees.
- Torso capsules were rotated about the wrong axis, so shoulders ran across the stance instead of along it.
- Blind numeric pose tweaking wasted iterations. Rendering the rig from four fixed orthogonal angles (back / side / front-3/4 / top) made every problem obvious in one pass.

## The five

| # | Name | Treatment |
|---|---|---|
| 1 | **Ink** | Near-black throughout, one coral accent on scarf/collar/gloves |
| 2 | Ember | Ink plus a warm back panel and board deck |
| 3 | Halo | Cool rim light baked into vertex colours, pale scarf |
| 4 | Strata | Three value steps — lighter helmet, mid board, gold accent |
| 5 | Signal | Near-black with saturated teal on goggles, scarf, gloves, deck |

## Outcome

**Ink shipped.** Cleanest silhouette and closest to the Alto's reference; the
single accent reads at chase distance without competing with the snow.

Not chosen, worth keeping on file:
- **Halo** was the most distinctive — the baked rim reads as lit-from-behind and holds up best against dark night snow. The rim is a vertex-colour bake against a fixed light direction, so it does not follow the day cycle; that would need a shader.
- **Signal** pops hardest but risks reading sci-fi against a nature palette.

Ink's skin is a five-colour object in the rig block. Switching treatments is a
data change, not a code change — the rim variant additionally needs the
vertex-colour bake pass restored.
