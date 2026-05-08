---
title: "HOMER (Hand-centered Object Manipulation Extending Ray-casting)"
sources:
  - "[[raw/articles/Advantages of Velocity-Based Scaling for Distant 3D Manipulation]]"
tags:
  - xr
  - vr
  - 3d-interaction
  - object-manipulation
  - distant-manipulation
  - ray-casting
  - hand-centered
created: 2026-05-08
updated: 2026-05-08
type: entity
---

**HOMER** (Hand-centered Object Manipulation Extending Ray-casting) is a hybrid 3D manipulation technique introduced by Bowman, Johnson, and Hodges (VRST '99). It combines two primitives that were previously used in isolation: **ray-casting for selection**, then **virtual-hand for manipulation**. After the user hits an object with a casted ray, the virtual hand "jumps" out to that object's depth and stays attached to it; subsequent hand motion translates the object directly. This is now the canonical baseline for at-a-distance manipulation in immersive VEs.

## Mechanics

At selection time, HOMER snapshots two distances (with the user's torso as origin):

- $D_\text{hand}$ — torso-to-physical-hand distance.
- $D_\text{object}$ — torso-to-selected-object distance.

A scale factor $D_\text{object}/D_\text{hand}$ is fixed for the duration of the grab. During manipulation:

$$
D_\text{virthand} = D_\text{currhand}\cdot \frac{D_\text{object}}{D_\text{hand}}
$$

The virtual hand position is reconstructed by walking $D_\text{virthand}$ along the **torso → hand ray**. To handle the case where the physical hand wasn't pointing exactly at the object at selection, HOMER also records an **offset vector** $V_\text{offset}$ (object's actual initial position minus its expected position on the body-hand ray) and adds it back during manipulation:

$$
P_\text{object} = D_\text{virthand}\cdot \frac{P_\text{hand} - P_\text{torso}}{D_\text{currhand}} + V_\text{offset}
$$

Object orientation is tied directly to hand orientation (rotations relative to the orientation at selection time).

## Why people like it

- **Both reach and rotation in one technique.** The selection ray reaches arbitrarily far; the hand-centered manipulation phase keeps the object directly attached to the hand for translate/rotate.
- **Pull-it-close gesture.** Bringing the physical hand toward the body multiplies through to a much larger reduction in $D_\text{virthand}$, so distant objects can be drawn into reach with a small motion. Conceptually similar to the AVP "z-axis telescoping" drag.
- **Body-relative reposition.** Because the virtual hand follows the torso-hand ray, simply *turning the body* swings the held object across the scene without arm motion.
- **Empirically efficient.** Established as a baseline in Bowman 1999 testbed evaluation and reaffirmed across VR manipulation studies.

## Why it's not enough

- **Reach ceiling.** $D_\text{virthand}$ can never exceed an arm-length scaled by $D_\text{object}/D_\text{hand}$. To move an object *further* than where it started, the user has to anticipate it at selection time by selecting with an arm-extended pose.
- **Jitter amplified.** Hand position is multiplied by the (often >1) ratio. Tracker jitter and hand tremor are amplified accordingly. Distant placement becomes near-impossible at any precision.
- **No CD-gain knob.** The mapping is constant for the duration of the grab; the user can't switch between coarse and fine without releasing and re-grabbing in a new pose.

The successor [[XR - Scaled HOMER]] addresses points 2 and 3 (and partially 1) by composing HOMER with a velocity-based CD-gain function. See [[XR - Scaled HOMER]] for the algorithm.

## Place in the lineage

HOMER is the workhorse of [[XR - Distant Object Manipulation|distant object manipulation]] alongside arm-extension techniques (Go-Go, Poupyrev 1996), world-scaling techniques (World in Miniature, Voodoo Dolls, Scaled World Grab), and pure ray-casting. Its hybrid design — ray to select, hand to manipulate — is also the implicit shape of Apple Vision Pro's gaze + pinch (gaze ≈ ray for selection, pinch + drag ≈ virtual hand for manipulation), one of the reasons HOMER's lessons keep transferring.

## Related

- [[XR - Scaled HOMER]] — velocity-based CD-gain extension.
- [[XR - Distant Object Manipulation]] — broader topic.
