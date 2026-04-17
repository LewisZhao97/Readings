---
title: "World Mouse: Exploring Interactions with a Cross-Reality Cursor — Summary"
sources:
  - "[[raw/articles/World Mouse Exploring Interactions with a Cross-Reality Cursor]]"
tags:
  - xr
  - interaction
  - input-devices
  - cursor
  - cross-reality
  - scene-understanding
  - google
created: 2026-04-17
updated: 2026-04-17
type: summary
recommend: partial
distil_times: 0
---

# World Mouse — Summary

Tütüncü, Gonzalez-Franco, Patel, Gonzalez (Google + U. Barcelona, 2026, arXiv 2603.10984, CHI '26). Position paper + prototype exploration arguing that the humble 2D desktop mouse, adapted correctly, is a better spatial input than raycasting or gaze+pinch for precision, low-fatigue XR work. Introduces the **World Mouse**: a cross-reality cursor that treats physical and virtual objects as one continuous interaction surface via semantic segmentation + mesh reconstruction.

## Key points

- **Framing / problem:** XR is over-invested in mid-air hand rays and gaze+pinch — both fatiguing and imprecise for long sessions or fine tasks (3D modeling, data annotation). Meanwhile many XR setups are already seated at desks. The mouse's precision-via-indirection is under-exploited.
- **Two core mechanisms:**
  1. **Within-object interaction** — surface-normal-aligned cursor tracking that lets the 2D mouse glide over the 3D surface of any single object (rasterization-like logic).
  2. **Between-object navigation** — an "invisible mesh" built from convex hulls of detected objects; Voronoi-style interpolation lets the cursor traverse empty space between disjoint objects fluidly.
- **Key extension over prior work:** the In-Depth Mouse [Zhou et al. 2022] and Everywhere Cursor [Kim & Vogel 2022] each handled a single modality (VR-only or AR-only). World Mouse unifies them — physical geometry from Meta Scene API / Android XR Scene Meshing is merged with virtual-asset geometry into a blended scene graph. Semantic segmentation (SAM-class models) labels physical objects so they become addressable, not just collidable.
- **2D→3D cursor transition:** explicit design for the moment the cursor leaves a 2D panel (e.g. an image editor window) and enters the room — builds on HandOver (UIST '25, same first author).
- **Prototype scenarios** (not a user study — vision-paper demos):
  - Desktop metaphor in 3D: point-and-click, radial right-click menus anchored to cursor, scroll-wheel repurposed for depth/zoom/scale.
  - 3D authoring: ghost-attached object placement, three-axis gizmos, spline editing, vertex-snapping to real-world meshes.
  - Cross-reality / IoT: virtual windows placed on real walls; interactive proxies on physical devices; "re-rendered reality" filters (e.g. passthrough sim of lights-off).
  - Cross-device: smartphone / smartwatch touchscreen as the World Mouse controller via XDTK (same group's prior toolkit).
  - AI grounding: precise cursor is proposed as the *deictic reference* mechanism for XR agents — unambiguously pinning "this object" without the fuzziness of gaze or voice.
- **Position vis-à-vis embodied interaction:** explicitly framed as *complementary*, not replacement. Freehand sketching / sculpting still belong to mid-air. Mouse wins on selection, confirmation, precision at varied depths.
- **What's absent:** no formal user study, no quantitative performance numbers, no implementation-detail deep dive on depth inference math (just cites In-Depth Mouse for the ray-projection + Voronoi-depth backbone). Headset hardware not specified (mentions Android XR Scene Meshing and Meta Scene API as candidate backends — implementation-agnostic).

## Entities mentioned

- **Authors / Google group:** Esen K. Tütüncü, Mar Gonzalez-Franco, Khushman Patel, Eric J. Gonzalez. Gonzalez-Franco's group is the thread running through much of this work — HandOver, PinchCatcher, XDTK, XR-Objects, EmBARDiment, Guidelines for Productivity in VR, SnapMove, AvatarPilot, RestfulRaycast, ForcePinch, Gaze+Pinch design principles all appear in the cites.
- **Systems / prior interaction techniques:** In-Depth Mouse (Zhou et al. 2022, CHI — the direct technical precursor), Everywhere Cursor (Kim & Vogel 2022), HandOver (UIST '25), XDTK cross-device toolkit, XR-Objects, EmBARDiment, HybridPointing, Go-Go, Erg-O, PRISM, SnapMove, Pinpointing, NanoStylus, Hyve-3D.
- **Tech backends:** Meta Scene API, Android XR Scene Meshing, Segment Anything (SAM), Diffuse Attend and Segment.
- **Concepts:** gaze+pinch, raycasting, gorilla arm, control-display (CD) gain, blended scene graph.

## Related topics (existing wiki)

- [[XR - AI + XR Integration]] — §4.5 "Interacting with AI" directly argues cursor-based precision as the deictic grounding layer for XR agents; fits the topic page's "Reality Model + agents" framing well.
- [[XR - XR Blocks]] — XR Blocks' `user.isSelectingAt(object)` intent grammar is precisely the kind of abstraction World Mouse's scene graph would feed. Cross-link worthwhile.

## Potential new wiki pages

- **Entity:** `XR - World Mouse` — the cursor system, the two core mechanisms, prototype catalogue, and position vs embodied interaction.
- **Topic:** `XR - Spatial Input & Pointing` — broader area encompassing raycasting, gaze+pinch, micro-gestures, cursors, control-display gain techniques. Several adjacent sources in cites; could accumulate fast.
- **Glossary additions:** Gorilla Arm, Control-Display (CD) Gain, Gaze+Pinch, Raycasting (XR), Scene Graph (blended), Deictic Reference.
- **Comparison candidate:** Cursor vs Raycasting vs Gaze+Pinch in XR — natural, would synthesize across World Mouse + prior ingests if any.

## Recommend Reason

**`partial`** — distil selectively. This is a **vision / prototype-exploration paper**, not a measured system like Vibe Coding XR. No numbers, no user study. But:

1. **Worth a small entity page** for World Mouse itself — it's a crisp, well-named technique that shows up in citations going forward, especially around XR-agent deixis.
2. **Worth a short topic-page extension** (or a new `XR - Spatial Input & Pointing` topic) — the paper's background section §2 is a mini-survey of CD-gain techniques (Go-Go, Erg-O, PRISM, SnapMove, pinpointing, gaze+pinch, micro-gestures) that cross-refs many related concepts. If future XR-input ingests arrive, having a topic scaffold helps.
3. **Worth adding to [[XR - AI + XR Integration]]** as a "spatial deixis for AI agents" datapoint — complements the Vibe Coding / XR Blocks agent-interface material.

Skip-list for distil: the cross-device smartphone/smartwatch section (§4.4) is mostly an advertorial for XDTK; the Discussion is philosophical ("thinking tool in the age of spatial computing") — no distillable claims. Focus distil on §2 (background mini-survey), §3 (two mechanisms + blended scene graph), §4.5 (AI deixis argument).
