---
type: Journal
date: 2026-07-27
session: trick-chain
description: Added rock bounces, tree-crown springs and grindable logs feeding a trick chain; platform-appropriate crash prompt.
status: shipped
related: [[decisions/2026-07-27-trick-chain]], [[physics]], [[mobile]]
---

# Trick chain + grinds

## Context
Tucker: the crash prompt's Y/N is meaningless on touch; and now that landings
work, add trick stacking — rock bounces, tree-crown springs, grindable logs.

## What changed
- Crash prompt adapts: touch gets "Keep going" / "End run", desktop reads "Keep going? Y / N".
- Rock crowns bounce, tree crowns spring, both adding a chain link.
- Logs grind, chaining a link every 0.25s, with a mandatory exit before the end.
- Chain banks on a clean landing into the multiplier plus a length-scaled bonus.

[[decisions/2026-07-27-trick-chain]]

## Measured
| behaviour | result |
|---|---|
| rock crown drop | 6/6 launched, chain +1, ~8.5 u/s |
| tree crown clip | 6/6 launched, chain +1, 12.5 u/s |
| rock at snow level | 5/5 still crash |
| tree trunk | 5/5 still crash |
| chain → multiplier | banks correctly, mult climbed 2 → 3 → 4 |
| grind | engages, chains, survives an intentional exit |

## Two bugs found by testing, not by looking
**Point tests miss fast crossings.** First implementation checked height only at
the frame's end position. At speed the rider crosses a crown inside one frame, so
the check read "below the crown" and crashed. Rock bounces fired 1 time in 6.
Fixed by testing the height band swept during the step.

**A launcher without a cooldown compounds.** Contact persisted across frames, so
one rock re-launched every frame: `chain=4, peakVy=33` — roughly a 17m rocket off
a knee-high rock. A 0.28s pass-through after any spring fixes it.

Both were invisible in a screenshot and only showed up in numbers.

## Open threads
- [ ] Tucker to play: are crowns too generous? The band is deliberately forgiving.
- [ ] Grind engagement is ~40% under a crude bot; unknown for a human aiming at it.
- [ ] `chain` feeds `mult`, which caps at 9, so long chains saturate fast. May want its own scale.
- [ ] Barrel rolls still unreachable — nothing sets `spinAxis` while ramps are off.

## Commit log

### 17:46 — d69d71e
Document the trick chain
files: knowledge/decisions/2026-07-27-trick-chain.md, knowledge/decisions/index.md, knowledge/journal/2026-07-27-mobile-and-ramps.md, knowledge/journal/2026-07-27-trick-chain.md, knowledge/journal/index.md, knowledge/wiki/physics.md, knowledge/wiki/roadmap.md

### 18:17 — 34f5de2
Log hop assist, and fix tilt steering at any hold angle
files: index.html

### 18:20 — ef27d40
Rewrite README as a game intro
files: README.md, knowledge/journal/2026-07-27-trick-chain.md, media/gameplay.gif, media/key-art.jpg, media/shot-dawn.jpg, media/shot-day.jpg, media/shot-golden.jpg, media/shot-night.jpg
