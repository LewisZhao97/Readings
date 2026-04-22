---
title: "Medial Axis Aware Learning of Signed Distance Functions — Summary"
sources:
  - "[[raw/articles/Medial Axis Aware Learning of Signed Distance Functions]]"
tags:
  - neural-sdf
  - signed-distance-function
  - medial-axis
  - phase-field
  - ambrosio-tortorelli
  - eikonal-equation
  - geometry-processing
  - surface-reconstruction
created: 2026-04-22
updated: 2026-04-22
type: summary
recommend: yes
distil_times: 2
---

## Overview

Weidemaier, Norden-Smoch, and Rumpf (Bonn) propose a variational neural method that jointly learns a signed distance function (SDF) and its **medial axis** from an unoriented point cloud. Two networks — a quadratic-MLP SDF $\phi_\theta$ and a SIREN-ResNet phase field $v_\eta$ — are trained simultaneously under an Ambrosio–Tortorelli-type loss that switches off a second-order normal-direction regularizer exactly on the gradient jump set. The result is competitive surface reconstruction *and* state-of-the-art global SDF accuracy (far-field distance), where prior methods typically excel at one or the other.

## Key Points

- **Problem framing.** The SDF is the viscosity solution of the eikonal equation $\|\nabla\phi\|=1$. Its gradient *must* jump on the medial axis (set where nearest-point projection is multi-valued), so any global higher-order regularizer must be inactive there.
- **Three-component loss + jump-set control.** (1) Eikonal penalty $\big|\|\nabla\phi\|^2-1\big|^2$; (2) higher-order term $v^2\|D^2\phi\,\nabla\phi\|^2 + \varepsilon^2\|D^2\phi\|^2$ (the relation $D^2\phi\,\nabla\phi=0$ encodes linear growth in normal direction); (3) Ambrosio–Tortorelli phase-field loss $\varepsilon\|\nabla v\|^2 + \tfrac{1}{4\varepsilon}(v-1)^2$ whose $\varepsilon\to 0$ limit recovers the $(d-1)$-Hausdorff measure of $J_{\nabla\phi}$.
- **Phase field as soft mask.** $v\approx 1$ where $\phi$ is smooth; $v$ collapses in an $\varepsilon$-band around the medial axis, locally disabling the second-order regularizer instead of fighting the unavoidable Hessian singularity.
- **Viscosity preference.** An exponential penalty $\sum_p \int \exp(-\alpha_p|\phi|^p)$ (with $\alpha_1=\alpha_2=100,\alpha_3=10$) pushes $|\phi|$ to grow, biasing toward the maximal (viscosity) solution among all a.e. eikonal solutions.
- **Reconstruction loss** $\tfrac{1}{\varepsilon^2}\int_\mathcal{S}\phi^2$ on the input point cloud, with the local weighting scheme from HeatSDF.
- **Architectures.** SDF: QuaNet — 4 layers (2D) / 8 layers (3D), 256 hidden units, quadratic pre-activation expansion + SIREN. Phase field: 4 (2D) / 6 (3D) ResNet blocks of 64/128 units, SIREN activations, sigmoid-clamped output. Adam, separate LRs ($5\!\times\!10^{-5}$ SDF, $5\!\times\!10^{-4}$ PF). $\varepsilon=10^{-3}$ in 2D, $10^{-4}$ in 3D.
- **3-phase training schedule.** Phase 1: SDF init from unit-sphere SDF with $v\equiv 1$, high $\gamma_\text{exp}$ to grow far field. Phase 2 (epoch 5+): jointly train PF, ramp $\gamma_\text{eik}$ and $\gamma_\text{HO}$ up, $\gamma_\text{exp}$ down. Phase 3 (epoch 20+): freeze PF, refine SDF, evaluate HO term on point cloud. 30 epochs total.
- **Adaptive Monte Carlo sampling (3D).** Local grid refinement in cells where $|\phi_\theta|<2^{-i}\tau_\text{sdf}$ or $v_\eta<\tau_\text{pf}$ ($\tau_\text{sdf}=0.1$, $\tau_\text{pf}=0.75$), with $(2^{id}n)^{-1}h^d$ MC weights — concentrates samples near surface and medial axis.
- **Theory.** Theorem 3.1: existence of minimizing tuples $(\phi^\varepsilon,v^\varepsilon)\in H^2\times H^1$ for fixed $\varepsilon>0$. A recovery-sequence construction (mollification on a tubular neighborhood of $J_{\nabla\phi}$) shows the loss components converge to their continuum limits; the matching $\liminf$ inequality (full $\Gamma$-convergence) is left open.
- **Results.** On SRB, ranks at or near top across $d_C, d_H, E_n$ (≈ tied with Hessian/HotSpot on Chamfer; best on $E_n$). Decisive lead on global SDF accuracy: $E_\text{eik}^\Omega = 0.0106$ vs. 0.04–0.77 for baselines; $E_\text{SDF}^\mathcal{N} = 0.0103$ best in class. ~85³ DoF, ~11 min/shape on a single A100 — competitive runtime despite two networks and second derivatives. Also evaluated on 40 Thingy10k shapes (sculpture/scan-heavy).
- **Sphere tracing.** Demonstrated on torus and hand shapes, fewer iterations per pixel than Hessian/HotSpot at comparable quality — direct payoff of accurate far-field SDF.
- **Limitations.** Co-dimension-2 medial-axis components (e.g., torus rotation axis) are not penalized by AT and show up as artifacts. Very complex topologies need smaller $\varepsilon$ + bigger nets and currently fail to converge with the fixed hyperparameter set. No analytical guarantee that $\mathcal{L}_\text{exp}$ actually selects the viscosity solution.
- **Plug-in usability.** Authors demonstrate the HO + AT regularizer dropped into HeatSDF's second step yields sharper level sets and less surface noise — suggesting the medial-axis-aware loss is a generic add-on to other neural SDF pipelines.

## Entities Mentioned

- **People:** Samuel Weidemaier, Christoph Norden-Smoch, Martin Rumpf (Institute for Numerical Simulation, U. Bonn)
- **Methods compared:** SIREN, DiGS, PHASE, StEik, Hessian, HeatSDF, HotSpot, 1-Lip, GSD (generalized signed distance, grid-based)
- **Architectures/components:** QuaNet (quadratic neural network), SIREN activations, ResNet, Adam, Marching Cubes (512³ extraction), PyTorch
- **Mathematical frameworks:** Eikonal equation, viscosity solutions, Mumford–Shah functional, Ambrosio–Tortorelli phase field, Modica–Mortola, Aviles–Giga conjecture, $\Gamma$-convergence, Hausdorff measure
- **Datasets:** Surface Reconstruction Benchmark (SRB), Thingy10k, HotSpot 2D dataset

## Related Topics

- **Neural implicit surface representation** — sits in the SIREN/DiGS/StEik lineage of variational neural SDFs.
- **Geometry processing & medial axis transform** — first method to *jointly* learn SDF and a phase-field approximation of the medial axis.
- **Phase field methods / Mumford–Shah** — borrows the AT trick for jump-set approximation from image segmentation.
- **Sphere tracing & implicit-surface rendering** — accurate far-field SDF directly improves ray-marching efficiency, connecting to ray-marching/real-time rendering interests already in the wiki.
- **PDEs on surfaces, level-set methods, collision detection, CSG** — listed application surface for high-quality global SDFs.

## Recommend Reason

**Recommend: yes.** This paper is worth distilling for several reasons. First, it's a clean and unusual marriage of two well-developed ideas (eikonal-loss neural SDFs and Ambrosio–Tortorelli phase-field segmentation) that produces an empirical jump on the *global* SDF accuracy metric where most neural baselines are weak — that's a genuinely new capability, not just a tuning win. Second, the HeatSDF augmentation experiment frames the HO+AT regularizer as a drop-in module, which is exactly the kind of reusable building block that earns an entity page and cross-links into a broader Neural SDF topic. Third, the structural insight — *use a second network as a learned mask for where your geometric regularizer is allowed to fire* — generalizes well beyond SDFs and is a worthwhile concept entry. The medial-axis side also opens a new entity (medial axis transform) the wiki doesn't yet cover. Distil should produce: an entity page for the method, a topic page on Neural SDF Learning (collecting the SIREN/DiGS/StEik/Hessian/HotSpot/HeatSDF/1-Lip lineage already implicit in this paper's references), entity pages for QuaNet and Ambrosio–Tortorelli, a concept page for the medial axis, and glossary entries for eikonal equation, viscosity solution, phase field, and medial axis.
