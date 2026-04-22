---
title: "Neural SDF Learning"
sources:
  - "[[raw/articles/Medial Axis Aware Learning of Signed Distance Functions]]"
tags:
  - neural-sdf
  - signed-distance-function
  - implicit-neural-representation
  - eikonal-equation
  - geometry-processing
created: 2026-04-22
updated: 2026-04-22
type: topic
---

How do you train a neural network $\phi_\theta\colon\mathbb{R}^d\to\mathbb{R}$ so that its zero-level set is a target surface $\mathcal{S}$ *and* its values are the signed distance to $\mathcal{S}$ everywhere in the domain? This topic surveys the design space — losses, architectures, regularizers — and the methods this knowledge base has touched.

The target is the [[Math - Eikonal Equation|eikonal-equation]] viscosity solution. Two competing pressures shape every method:

- **Surface fidelity (near-field)** — the zero-level set should reconstruct $\mathcal{S}$ with detail, sharp edges, and correct topology. Measured by Chamfer distance, Hausdorff distance, normal alignment, narrow-band SDF error.
- **Distance fidelity (far-field)** — values away from the surface should equal true signed distance. Measured by global eikonal error and global SDF error. Required by sphere tracing, collision detection, level-set PDEs.

Most methods optimize one at the expense of the other. [[Graphics - Medial Axis Aware SDF Learning|Medial-Axis-Aware SDF]] is the first in this knowledge base to score top-tier on both simultaneously.

## The losses people use

| Loss | Form | Idea |
|------|------|------|
| Reconstruction | $\int_\mathcal{S}\phi^2$ | Anchor zero-level set on input point cloud. |
| Eikonal | $\int(\|\nabla\phi\|^2-1)^2$ or $\int\big|1-\|\nabla\phi\|\big|$ | Make $\phi$ a distance function. Non-convex in $\nabla\phi$. |
| Higher-order Hessian | $\int\|D^2\phi\|^2$ (Laplace, Frobenius) | Smoothness regularizer. |
| Normal-direction Hessian | $\int\|D^2\phi\,\nabla\phi\|^2$ | Encodes linear growth in normal direction (eikonal consequence). |
| Hessian determinant | $\int|\det D^2\phi|$ | Encodes singular-Hessian structure of the SDF. |
| Maximality / exponential | $\int e^{-\alpha\|\phi\|^p}$ | Bias toward viscosity (largest $|\phi|$). |
| Heat-equation coupling | $\phi \approx -t\log u_t$ | Use the heat solution as a smooth proxy for distance. |
| Phase-field jump-set ([[Math - Ambrosio-Tortorelli Phase Field|AT]]) | $\int\varepsilon\|\nabla v\|^2 + \tfrac{1}{4\varepsilon}(v-1)^2$ | Approximate $\mathcal{H}^{d-1}([[Graphics - Medial Axis|J_{\nabla\phi}]])$ with a smooth gating field. |

## Family tree (as cited by [[Graphics - Medial Axis Aware SDF Learning]])

| Method | Key idea | Input | Loss highlights |
|--------|----------|-------|-----------------|
| **SIREN** (Sitzmann 2020) | Sine-activated MLP + eikonal | Unoriented PC | Reconstruction + eikonal + exp growth |
| **PHASE** (Lipman 2021) | Modica–Mortola two-phase occupancy → log-transform to SDF | PC | Modica–Mortola; SDF only in postprocess |
| **DiGS** (Ben-Shabat 2022) | SIREN + Laplacian L² regularizer | Unoriented PC | Adds $\int\|\Delta\phi\|^2$ |
| **Neural-Singular-Hessian** (Wang 2023) | Near-field Hessian determinant | Unoriented PC | Hessian degeneracy in near field |
| **Hessian / Aligning Gradient & Hessian** (Wang 2023) | Near-field $D^2\phi\,\nabla\phi$ | Unoriented PC | Strong on Chamfer, weak on global eikonal |
| **StEik** (Yang 2023) | $L^1$ loss on directional divergence in normal direction | Unoriented PC | Stabilises eikonal optimisation |
| **HotSpot** (Wang 2025) | One-step SDF–heat coupling: $\phi\approx -t\log u_t$ | Unoriented PC | Heat–eikonal joint loss |
| **HeatSDF** (Weidemaier 2026) | Two-step: heat equation then SDF (Crane-style, neural) | Oriented PC | Best $E_\text{SDF}^\mathcal{N}$ pre-this-work; weak global eikonal |
| **1-Lip** (Coiffier & Béthune 2024) | Lipschitz-constrained network, distance-by-construction | Oriented PC | Slow; sometimes fails to converge |
| **GSD** (Feng & Crane 2024, grid-based) | Generalised heat-method SDF on grid | Oriented PC | 128³ grid baseline |
| **Medial-Axis-Aware** (Weidemaier 2026, this work) | Global $\|D^2\phi\,\nabla\phi\|^2$ + AT-learned medial-axis mask | Unoriented PC | Best global SDF + competitive surface |

Trend: methods have moved from purely first-order (SIREN, eikonal-only) to second-order regularizers exploiting the eikonal structural identity $D^2\phi\,\nabla\phi=0$ — first locally (in the near field, where the medial axis is far away), then globally with an explicit jump-set treatment.

## Architectural choices

- **Activations:** SIREN (sinusoidal) almost universal — encodes high-frequency detail and gives meaningful second derivatives.
- **Backbone:** plain MLP (SIREN, DiGS) → MLP with quadratic pre-activation (QuaNet, used by [[Graphics - Medial Axis Aware SDF Learning]] for $\phi_\theta$).
- **Twin networks:** Medial-Axis-Aware uses a separate ResNet for the phase field $v_\eta$. PHASE uses one network for occupancy. Most methods use a single SDF network.
- **Sampling:** uniform Monte Carlo → adaptive refinement near the surface and (in the medial-axis-aware case) near the predicted jump set, with importance weights $(2^{id}n)^{-1}h^d$.

## Open problems flagged by the source

- **Co-dimension-2 medial-axis components** are invisible to AT (which approximates $(d-1)$-measure). Axial-symmetric shapes get artifacts.
- **Complex topologies** require smaller phase-field $\varepsilon$ + bigger nets that currently fail to converge with standard schedules.
- **Viscosity-solution selection** by the exponential penalty has empirical support but no proof.
- **Full $\Gamma$-convergence** of the AT-style higher-order phase-field loss to its limit is open beyond the recovery direction.

## Connections in this wiki

- [[Math - Eikonal Equation]] — the PDE these methods solve.
- [[Math - Ambrosio-Tortorelli Phase Field]] — the regularizer that enables global second-order training.
- [[Graphics - Medial Axis]] — the geometric obstacle every method must accommodate.
- [[Graphics - Ray Marching]] — the consumer of well-conditioned far-field SDFs.
- [[Graphics - Neural Rendering]] — sister direction (replace the *renderer*, not the *geometry*); SDFs are sometimes the geometry inside a neural renderer.
