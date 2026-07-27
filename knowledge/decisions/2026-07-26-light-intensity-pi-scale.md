---
type: Decision
date: 2026-07-26
description: All ambient/hemisphere/directional intensities in the day table are ~PI times "intuitive" values, because three.js r155 removed the legacy light scaling.
status: accepted
deciders: Tucker
related: [[atmosphere]]
---

# Decision: Day-palette light intensities are PI-scaled

## Context
The first render of the scene came out roughly three times too dark — snow read
as brown-mauve at dawn despite intensities that looked reasonable.

## Decision
Author every `si` / `hi` / `ai` value in the `DAY` table at approximately π times
the "intuitive" value, and note why in a comment above the table.

## Consequences
- **Positive:** the scene is correctly exposed across all eight keyframes.
- **Negative:** the numbers look wrong to anyone carrying pre-r155 three.js intuition, hence the comment.
- **Neutral:** purely an authoring convention; no code path depends on it.

## Dissent / Alternatives Considered
- **Set `WebGLRenderer.useLegacyLights = true`** — the flag was deprecated in r155 and removed in r165. Pinning to it would block ever upgrading three.js.
- **Add tone mapping to lift the image** — would change the sky shader and sprite colours too, and makes the hand-picked palette unpredictable.

## Sources
- Diagnosed by measuring rendered pixel values against the Lambert BRDF, not by eye. [[journal/2026-07-26-build-and-review]]
