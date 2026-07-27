---
type: Decision
date: 2026-07-27
description: Wiping out offers three continues per run that carry score and distance; running out ends the run to a local top-5 leaderboard with a JPEG card export.
status: accepted
supersedes: instant auto-respawn with score reset to zero
deciders: Tucker
related: [[physics]]
---

# Decision: Three continues per run, then a leaderboard

## Context
Originally a crash reset the score to zero and auto-respawned after 1.5 s.
Tucker wanted a "keep going? Y/N" prompt with three seconds of invincibility on
continue, and an exit to a leaderboard downloadable as a JPEG.

Free unlimited continues would mean the run never ends and the leaderboard means
nothing — so the cost of a continue was the open question. Tucker chose a limit
of three.

## Decision
- Crash → 1.15 s tumble → dimmed panel: score, distance, best multiplier, "Keep going? N left."
- **Y** — continue in place, score and distance carry, multiplier resets to 1, 3.0 s of collision grace.
- **N**, or continues exhausted — end the run, record it, show a local top-5 leaderboard with "Ride again" and "Save card."
- The card is a 1080×1350 canvas rendered from the title screen's dawn palette, exported as JPEG.

## Consequences
- **Positive:** a run is now a bounded thing with a real end, so leaderboard entries are comparable to each other.
- **Positive:** continues preserve flow after a cheap death without erasing the session.
- **Negative:** scores are no longer comparable to any pre-change best — a session now accumulates across up to four lives and reaches ~1,700 m where it used to reach ~200 m.
- **Neutral:** the leaderboard is `localStorage` only. No accounts, no server.

## Dissent / Alternatives Considered
- **Free and unlimited continues** — offered; rejected because it removes the fail state entirely and makes the leaderboard a session-length record.
- **Free continue but score resets** — offered; preserves position and flow but not points. Rejected as the weaker of the two.

## Sources
- Answered directly by Tucker when asked. [[journal/2026-07-27-feel-rework]]
