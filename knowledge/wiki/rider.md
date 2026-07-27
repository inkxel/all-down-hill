---
name: rider
description: The rigged snowboarder — procedural geometry, a two-bone IK leg solve, and animation driven entirely from game state.
type: subsystem
created: 2026-07-27
last_updated: 2026-07-27
confidence: high
related: [[physics]], [[index-map]], [[research/2026-07-rider-comps]]
---

# Rider — rig and animation

## Hierarchy

```
player      world position, terrain-normal alignment + yaw
 └ bank     roll: carve lean + barrel-roll rotation
    └ flip  pitch: backflip, pivoting at y 0.82 (the rider's mass, not the board)
       └ body
          └ root (scaled 1.15)
             ├ board, bindings, boots        static
             └ pelvis → hips → knees         IK
                  └ spine → chest → shoulders → elbows → gloves
                                 └ neck → head, helmet, goggles
             └ scarf                          hung off root, not chest
```

The scarf is deliberately parented to `root` rather than `chest`: parented to the
torso, the body's carve twist swung it sideways across the silhouette.

## Geometry

All procedural, built once at startup:
- **Board** — a bezier `Shape` extruded with a bevel, then nose/tail rocker applied by displacing vertices as `|z|^2.8`.
- **Bones** — `CapsuleGeometry` translated down by half its length so it hangs from its own origin, which lets joints chain without offset objects.
- **Scarf** — a swept tube with a twisting frame. A flat ribbon was tried first and vanished edge-on from the chase camera.

## IK legs

`ik2` is a standard two-bone solve: law of cosines for the hip and knee angles,
then an explicit orthonormal basis built from the hip→ankle direction and a bend
hint pointing toward the toe edge. `makeBasis` + `setFromRotationMatrix`.

The payoff is that **crouch depth is a free parameter** — the ankles stay locked
to the bindings whatever the pelvis does, so landing compression and air tuck
animate by moving one number.

## Animation inputs

`poseRider(dt, turn, air, compress)` — every value is damped, nothing is keyframed:

| Input | Drives |
|---|---|
| `compress` | crouch depth (spikes on landing impact, decays) |
| `air` | tuck: taller stance, arms pulled in |
| `turn` (`vx / STEER_MAX`) | torso twist, lean, arm swing, scarf sweep |

## Skin

Rider 1, "Ink" — near-black body and board, one coral accent on scarf, collar and
gloves. Chosen from a five-up comp sheet; see [[research/2026-07-rider-comps]] for
the alternatives and why they were rejected.

## Context log

### 2026-07-27 — Article created
Replaced the original box-primitive rider. Tucker asked to ship this and give
feel notes afterwards, so the pose constants are explicitly provisional.
[[journal/2026-07-27-feel-rework]]
