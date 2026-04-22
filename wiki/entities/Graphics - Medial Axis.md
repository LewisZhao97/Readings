---
title: "Medial Axis"
sources:
  - "[[raw/articles/Medial Axis Aware Learning of Signed Distance Functions]]"
tags:
  - geometry-processing
  - medial-axis
  - signed-distance-function
created: 2026-04-22
updated: 2026-04-22
type: entity
---

The **medial axis** of a surface $\mathcal{S}\subset\mathbb{R}^d$ is the closure of the set of points where the nearest-point projection onto $\mathcal{S}$ is not single-valued — equivalently, points equidistant from at least two points of the surface. For a smooth closed surface in $\mathbb{R}^d$ it is a lower-dimensional set (typically a $(d-1)$-dimensional skeleton with $(d-2)$-dimensional creases).

## Why it matters for SDFs

The signed distance function $\phi=\mathrm{sgndist}(\cdot,\mathcal{S})$ is Lipschitz and differentiable almost everywhere, with $\|\nabla\phi\|=1$ wherever it is differentiable. But the gradient *must* jump on the medial axis: as you cross it, the nearest surface point switches discontinuously, so the gradient direction (which points away from that nearest point) flips. This jump set is denoted $J_{\nabla\phi}$, and it coincides with the medial axis even for smooth surfaces.

Two structural consequences:

1. **No globally smooth SDF exists** for any closed surface — gradient discontinuities are intrinsic, not noise.
2. **Higher-order regularizers must be masked.** Any loss involving $D^2\phi$ must switch off in a neighborhood of the medial axis or it will fight an unavoidable singularity. Prior neural SDF methods (Hessian, Neural-Singular-Hessian) handled this implicitly by restricting to the *near field* of the surface; [[Graphics - Medial Axis Aware SDF Learning]] does it explicitly via a learned phase field.

## Identification via [[Math - Ambrosio-Tortorelli Phase Field]]

The Ambrosio–Tortorelli loss
$$\int_\Omega \varepsilon\|\nabla v\|^2 + \tfrac{1}{4\varepsilon}(v-1)^2 \mathrm{d}x$$
applied to a function $v\in[0,1]$ that gates a Dirichlet-like term in $\phi$, produces (in the $\varepsilon\to 0$ limit) a $v\equiv 1$ background with a thin transition where the gated term wants $v\to 0$. The $L^1$ measure of the transition recovers the $(d-1)$-Hausdorff measure of the jump set $J_{\nabla\phi}$. So minimizing AT alongside an HO term that wants to be active everywhere except on $J_{\nabla\phi}$ teaches $v$ to localize on the medial axis.

## Limitations of the AT approach

The AT energy approximates a $(d-1)$-dimensional measure. **Co-dimension-2 components** of the medial axis (e.g., the rotation axis of a torus, or sharp creases of $J_{\nabla\phi}$ in 3D) carry zero $(d-1)$-measure and so escape penalization. They still appear in $v_\eta$ but are not weighted appropriately in the loss, leading to artifacts in shapes with axial symmetry.

## Applications listed in the source

- Collision detection between SDF-represented objects.
- Constructive solid geometry on neural SDFs.
- Level-set methods for PDEs on surfaces.
- Sphere tracing / implicit-surface ray marching — accurate far-field distances (which depend on a faithful medial-axis structure) directly reduce step counts. See [[Graphics - Ray Marching]].

## Related concepts

- [[Math - Eikonal Equation]] — the medial axis is exactly the gradient jump set of the viscosity solution.
- [[Graphics - Neural SDF Learning]] — methods that have to confront medial-axis singularities.
