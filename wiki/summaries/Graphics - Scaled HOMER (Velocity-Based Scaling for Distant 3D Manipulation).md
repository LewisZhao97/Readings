---
title: "Advantages of Velocity-Based Scaling for Distant 3D Manipulation — Summary"
sources:
  - "[[raw/articles/Advantages of Velocity-Based Scaling for Distant 3D Manipulation]]"
tags:
  - xr
  - vr
  - 3d-interaction
  - object-manipulation
  - distant-manipulation
  - homer
  - scaled-homer
  - velocity-based-scaling
  - control-display-gain
  - prism
  - vrst-2008
created: 2026-05-08
updated: 2026-05-08
type: summary
recommend: yes
distil_times: 1
---

## Overview

Wilkes and Bowman (Virginia Tech, VRST '08) extend the PRISM principle of *speed-adaptive control-display gain* to **HOMER**, the canonical ray-cast-then-virtual-hand technique for at-a-distance 3D manipulation. The resulting **Scaled HOMER** preserves the look-and-feel of HOMER for coarse motion but quietly scales hand displacement down when the user moves slowly, regaining fine-grained precision that HOMER's distance-amplified mapping otherwise destroys. A 14-participant CAVE study finds Scaled HOMER nearly halves task time over HOMER (5.17 s vs. 9.93 s, p < 0.001), with the largest gains exactly where HOMER hurts most: distant targets, small targets, and long movements.

## Key Points

- **Problem framing.** Distant 3D manipulation requires *amplifying* the input mapping (so the user can reach across the scene with a short arm motion) but amplification multiplies tracker jitter and hand tremor, making precision impossible at distance. Wilkes & Bowman argue you need both: scale up for reach, scale down for precision, in one technique.
- **HOMER recap.** HOMER (Bowman 1999) selects via ray-cast, then attaches the object to a virtual hand that lives on the body→hand ray at distance $D_\text{virthand} = D_\text{currhand}\cdot(D_\text{object}/D_\text{hand})$, with an offset vector preserving the original object position relative to that ray. Hand rotation maps directly to object rotation. The torso-anchored ray means large reorientations of the user move objects by huge arcs, but small arm motions are scaled up by the (often >1) ratio $D_\text{object}/D_\text{hand}$ — and so is jitter.
- **Velocity-based scaling function.** Per frame, compute physical hand displacement $D_\text{hand}$ and velocity $V_\text{hand}$. The scaled distance is $SD_\text{hand} = \min(V_\text{hand}/SC,\ 1.2)\cdot D_\text{hand}$ where $SC$ is a fixed velocity threshold. So when $V_\text{hand} = SC$ the scaling factor is exactly 1.0; below that, motion is damped; above it, motion is *amplified* up to a 1.2× cap. A minimum-velocity floor zeroes $SD_\text{hand}$ entirely, freezing the object during pure tremor.
  - The 1.2× upper bound was set by pilot tests — anything more felt unmanageable.
  - The cap addresses HOMER's *reach* limitation (your virtual hand can't go beyond an arm-extension scaled by $D_\text{object}/D_\text{hand}$): a brief faster-than-SC motion still buys a small amount of extra reach.
- **Composing the two scalings.** First-attempt design multiplied the velocity scale and the HOMER scale into a single vector — failed: the object stopped tracking the body-hand ray and "felt" wrong. Working design **applies velocity scaling first to obtain a virtual hand position $SP_\text{hand}$**, then feeds that into the unmodified HOMER equation. This decoupling is presented as a general recipe: any manipulation technique that consumes a hand position can be wrapped in PRISM-style velocity scaling without re-deriving its math.
- **"Same feel" design goal.** Users familiar with HOMER reportedly couldn't distinguish Scaled HOMER from HOMER until they attempted a fine-grained motion. Scaled HOMER deliberately becomes invisible during coarse interaction; only precision tasks reveal it.
- **Object trajectory difference.** In HOMER the object stays exactly on the torso-hand ray. In Scaled HOMER it stays on the ray when $V_\text{hand} = SC$ but drifts off it when the user moves slowly (because slow motion shrinks $SD_\text{hand}$ and the object lags). Authors argue this is rarely noticeable since (a) it only happens at low velocity and (b) HOMER itself can leave the object off the ray via the offset vector.

## Experiment

- **Apparatus.** 4-screen CAVE (10×10×9 ft, stereo via LC shutter glasses). Intersense IS-900 6-DoF tracking on head and dominant hand; "wand" with buttons + analog joystick.
- **Design.** Within-subjects, five IVs: technique (HOMER / Scaled HOMER), target size (small 0.38×0.46 m / medium 1.07×1.52 m), target distance from user (near 15.24 m / far 60.96 m), movement direction (left / toward user / away), movement distance (short 7.62 m / long 38.1 m). 24 conditions × 3 trials × 2 techniques = 144 trials per participant. 30-second cap per trial.
- **Participants.** 14 (1 female), undergraduate CS recruits. Order counterbalanced.
- **DV.** Task completion time (log-transformed for ANOVA).
- **Result.** Mean time HOMER 9.93 s vs. Scaled HOMER 5.17 s, F(1,531)=339.32, p<0.0001. All five main effects significant. Notable interactions:
  - **Technique × target distance** — Scaled HOMER's advantage grows at the far target (60.96 m), exactly where HOMER's amplification of jitter is worst.
  - **Technique × target size** — advantage grows on the small target, as expected for a precision-aimed technique.
  - **Three-way technique × target distance × movement distance** — Scaled HOMER also helps in the *near-target / long-movement* case, because the object's initial position (and thus HOMER's scale factor) was distant; that scale factor stays large during the motion.

## Entities Mentioned

- **People:** Curtis Wilkes, Doug A. Bowman (Virginia Tech).
- **Methods:** HOMER (Bowman 1999), PRISM (Frees, Kessler & Kay 2007), Scaled HOMER (this paper), Go-Go arm extension (Poupyrev 1996), ray-casting (Bowman & Hodges 1997), World in Miniature (Stoakley 1995), Voodoo Dolls (Pierce 1999), Scaled World Grab (Mine 1997), snap-dragging (Bier 1990).
- **Hardware / software:** CAVE (Cruz-Neira 1993), Intersense IS-900, DIVERSE framework (Kelso 2002), SGI Performer, C++.
- **Domain refs:** Mackenzie & Riddersma 1994 (CD-gain in desktop GUIs), Bowman et al. 2004 *3D User Interfaces: Theory and Practice* (textbook).

## Related Topics

- **XR direct manipulation lineage.** Sits squarely in the PRISM → Scaled HOMER → modern AVP/Quest CD-gain story. The "scale velocity first, hand it off to any manipulation technique" recipe is exactly the kind of factoring that informs how visionOS's z-axis telescoping and Quest's pinch-and-drag are tuned.
- **Distant object manipulation.** Direct ancestor of the BMSR / Scaled HOMER+NFSRV (IEEE VR 2024) and adaptive-gain (ISMAR 2022) work — both build on Scaled HOMER, both improve on its limits (no scaled-replica context, fixed scaling curve).
- **Control-display gain in mid-air interaction.** Connects to the recent CHI '25 *Everything to Gain* line in touchless input — same idea, applied to flat displays.
- **Eye+hand interaction (gaze + pinch).** Pfeuffer's *Design Principles & Issues for Gaze and Pinch* (2024) explicitly raises the CD-ratio problem this paper solves; AVP's drag UX is a (proprietary) successor in the same family.

## Recommend Reason

**Recommend: yes.** Short, practical, and exactly the algorithm the user is trying to understand. Three reasons to distil:

1. **It's the cleanest published statement of the speed-adaptive CD-gain pattern for distant XR manipulation.** PRISM is the textbook reference, but PRISM is virtual-hand-only and stays in arm's reach. Scaled HOMER is the *distant* version — relevant to AVP's "drag a window 3 metres away" and Quest's pinch-drag at any depth.
2. **The compositional recipe** ("apply scaling to the hand position, then feed it to whatever distant-manipulation technique you have") is the design pattern the user should walk away with. It generalizes — it's how you would bolt CD-gain onto a custom XR runtime drag without rewriting the runtime's manipulation math.
3. **It opens the connections** in our XR-interaction subgraph: a new `Scaled HOMER` entity and a new `XR Distant Object Manipulation` topic page would tie together the PRISM, HOMER, Go-Go, BMSR, and gaze-and-pinch ancestry that the recent /feedme run surfaced. Several glossary entries are obvious: HOMER, Scaled HOMER, PRISM, control-display gain, velocity-based scaling, ray-casting (manipulation sense), Go-Go.

Distil should produce: an entity page for Scaled HOMER (this paper), an entity page for HOMER (the parent technique), and the start of a topic page on *XR Distant Object Manipulation* / *Control-Display Gain in XR* that this paper anchors and that subsequent /ingests of PRISM, the IEEE VR 2024 BMSR work, and Pfeuffer's gaze-and-pinch papers will fill in.
