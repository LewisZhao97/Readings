---
title: "Scaled HOMER (Velocity-Based Scaling for Distant 3D Manipulation)"
sources:
  - "[[raw/articles/Advantages of Velocity-Based Scaling for Distant 3D Manipulation]]"
tags:
  - xr
  - vr
  - 3d-interaction
  - distant-manipulation
  - scaled-homer
  - homer
  - velocity-based-scaling
  - control-display-gain
  - prism
created: 2026-05-08
updated: 2026-05-08
type: entity
---

**Scaled HOMER** (Wilkes & Bowman, ACM VRST '08) is a velocity-adaptive extension of [[XR - HOMER|HOMER]] that adds a **speed-dependent control-display gain** to the user's hand input. Slow hand → motion damped (precision); fast hand → motion preserved or briefly amplified up to 1.2× (reach). The technique is specifically aimed at the failure case HOMER never solved: precise placement of objects at large distances, where HOMER's $D_\text{object}/D_\text{hand}$ scaling amplifies jitter beyond what the user can correct.

## The velocity-based scaling function

Per frame:

$$
SD_\text{hand} = \min\!\left(\frac{V_\text{hand}}{SC},\ 1.2\right)\cdot D_\text{hand}
$$

where $V_\text{hand}$ is the current physical hand velocity, $D_\text{hand}$ is the per-frame physical displacement, and $SC$ is a fixed velocity threshold (the "neutral speed" at which the gain is exactly 1.0). Below $SC$ the gain shrinks linearly; above it the gain rises but is **capped at 1.2×**. Pilot tests showed any higher cap was unmanageable. A separate minimum-velocity floor zeroes $SD_\text{hand}$ entirely, freezing the object during pure tremor.

The cap >1 is doing real work: it partially mitigates HOMER's hard reach ceiling (you can extend slightly beyond the original arm-scaled limit by moving fast enough).

## The compositional recipe

A first-attempt design multiplied the velocity scale and the HOMER scale into a single output vector. It failed: objects no longer tracked the torso-hand ray, and the technique "felt" wrong.

The working design **applies velocity scaling to the physical hand position**, producing a *scaled hand position*

$$
SP_\text{hand} = SD_\text{hand}\cdot \frac{V_\text{hand,move}}{\|V_\text{hand,move}\|} + P_\text{hand}
$$

and feeds that into the unmodified HOMER equation:

$$
D_\text{virthand} = SP_\text{hand}\cdot \frac{D_\text{object}}{D_\text{hand}}
$$

This factoring is the durable contribution: **velocity-based CD gain becomes a wrapper over the input**, so any manipulation technique that consumes a hand position (HOMER, Go-Go, gaze + pinch, etc.) can be CD-gain-augmented without re-deriving its math. The recipe — *scale velocity → produce a synthetic hand position → hand off* — is reusable across the whole [[XR - Distant Object Manipulation|distant object manipulation]] family.

## "Same feel" design goal

A stated goal: users familiar with HOMER should not be able to tell Scaled HOMER apart until they attempt a precision motion. This was reportedly achieved — only fine-grained tasks revealed the difference. The technique is invisible during coarse interaction.

## Trajectory difference

In HOMER the held object stays exactly on the torso-hand ray. In Scaled HOMER it stays on the ray when $V_\text{hand}=SC$ (gain = 1.0), but drifts off it when the user moves slowly (because the damped motion lags the ray's swing). Authors argue this is rarely noticeable since (a) it only occurs at low velocity and (b) HOMER itself can already leave the object off the ray via the offset vector.

## Empirical result

CAVE study, n = 14, within-subjects 5-IV design (technique × target size × target distance × movement direction × movement distance), 144 trials per participant, 30 s cap.

| Variable | Scaled HOMER | HOMER | Effect |
|---|---|---|---|
| Mean task time | **5.17 s** | 9.93 s | F(1,531)=339, p<0.0001 |
| Far target (60.96 m) | strong advantage | — | technique × distance interaction, p<0.0001 |
| Small target (0.38×0.46 m) | strong advantage | — | technique × size interaction, p<0.0001 |
| Long movement (38.1 m) from a far origin | strong advantage | — | three-way interaction, p<0.0002 |

The pattern matches the design intent: gains concentrated where HOMER's amplified-jitter problem hurts most (far + small + long).

## What it doesn't do

- **No depth-aware adaptation.** The gain is purely velocity-driven; the gain ceiling 1.2 is a fixed constant rather than a function of $D_\text{object}/D_\text{hand}$. Later work (e.g., adaptive-gain ISMAR '22) couples gain to context.
- **No replica view.** The user still sees the object only at its world position. Compare BMSR / Scaled HOMER+NFSRV (IEEE VR '24), which augment HOMER with a near-field scaled replica of the target's *context*.
- **No automatic SC tuning.** Threshold $SC$ is hand-set; the paper doesn't analyse how to pick it per user / device.

## Place in the lineage

- **Predecessor:** [[XR - HOMER|HOMER]] (Bowman 1999) — selection ray + virtual hand at distance.
- **Sibling:** PRISM (Frees, Kessler & Kay, TOCHI 2007) — same speed-adaptive gain idea, virtual-hand-only / arm's-reach.
- **Synthesis (this work):** Scaled HOMER = PRISM ∘ HOMER, with the explicit recipe to compose CD-gain with any distant-manipulation technique.
- **Successors:** Adaptive-gain distant manipulation (ISMAR '22); BMSR + Scaled HOMER+NFSRV (IEEE VR '24); SightWarp (UIST '25). Modern XR runtimes (Apple Vision Pro, Meta Quest) implement de-facto CD-gain on pinch-drag whose tuning curves are direct descendants of this paper.

## Related

- [[XR - HOMER]] — the parent technique.
- [[XR - Distant Object Manipulation]] — broader topic, including the modern AVP / Quest descendants.
