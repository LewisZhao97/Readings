---
title: "Eikonal Equation"
sources:
  - "[[raw/articles/Medial Axis Aware Learning of Signed Distance Functions]]"
tags:
  - pde
  - eikonal-equation
  - viscosity-solution
  - signed-distance-function
created: 2026-04-22
updated: 2026-04-22
type: entity
---

The **eikonal equation** is the first-order Hamilton–Jacobi PDE
$$
\|\nabla\phi(x)\| = 1 \quad \text{a.e. in } \Omega,
$$
typically with boundary condition $\phi=0$ on a surface $\mathcal{S}\subset\Omega$. Its solutions describe distance-like functions whose level sets advance at unit speed in the normal direction.

## Solution structure

There are infinitely many a.e. solutions for a given $\mathcal{S}$ — any function built from piecewise-isometric "tents" with $\|\nabla\phi\|=1$ vanishing on $\mathcal{S}$ qualifies (see Fig. 3 of [[Graphics - Medial Axis Aware SDF Learning]] for a 1D illustration). Uniqueness is recovered by selecting the **viscosity solution**.

### Viscosity solution

The viscosity solution is the unique weak solution selected by a comparison principle. It coincides with the **signed distance function** $\mathrm{sgndist}(\cdot,\mathcal{S})$ and is *maximal* in absolute value among all a.e. solutions vanishing on $\mathcal{S}$. This maximality is what neural-SDF methods exploit when they add an exponential penalty $\sum_p\int\exp(-\alpha_p|\phi|^p)$ — it biases the optimizer toward the largest-magnitude solution. The bias is empirical; no proof yet links the penalty to viscosity selection. Note that minimising $\mathcal{H}^{d-1}(J_{\nabla\phi})$ alone does *not* uniquely select the viscosity solution either — [[Graphics - Medial Axis Aware SDF Learning]] Fig. 4 exhibits an a.e. eikonal solution with shorter jump-set measure than the viscosity solution.

### Linear growth in normal direction

Differentiating $\|\nabla\phi\|^2 = 1$ and using symmetry of the Hessian gives the structural identity
$$
D^2\phi\,\nabla\phi = 0
$$
wherever $\phi$ is twice differentiable. Geometrically: $\nabla\phi$ is constant along its own integral curves, i.e. the SDF grows linearly along rays normal to the level sets. This identity is the basis of higher-order neural-SDF regularizers (Hessian / Neural-Singular-Hessian use it in the near field; [[Graphics - Medial Axis Aware SDF Learning]] uses it globally with a [[Graphics - Medial Axis|medial-axis]] mask).

### Gradient jump set

Lipschitz a.e. solutions of the eikonal equation generically have a discontinuous gradient on a $(d-1)$-dimensional set $J_{\nabla\phi}$. For the viscosity solution this set is the **medial axis** of $\mathcal{S}$.

## Theoretical lineage — Aviles–Giga

Aviles and Giga (1987, *A mathematical problem related to the physical theory of liquid crystal configurations*) proposed a Modica–Mortola-type functional with *higher-order derivatives* and conjectured that its sharp-interface limit is a functional depending on the jump set of the gradient of an a.e. eikonal solution. This is exactly the object that [[Graphics - Medial Axis Aware SDF Learning]] targets through its HO + [[Math - Ambrosio-Tortorelli Phase Field|AT]] loss. In 2D, the limit space was identified by Ambrosio–De Lellis–Mantegazza (1999); the sharp lower bound by Jin–Kohn (2000); Aviles–Giga (1996) drew the connection to viscosity solutions. De Lellis (2002) showed higher-dimensional behavior differs fundamentally — the reason the 3D $\Gamma$-convergence is still open and why neural methods in this space only prove existence + recovery, not the full $\liminf$ inequality.

## Why it's hard for neural networks

- **Non-convex** in $\nabla\phi$ — first-order eikonal losses lack lower semi-continuity, so existence of minimizers is not guaranteed for first-order methods (one motivation for adding the convex second-order term in [[Graphics - Medial Axis Aware SDF Learning]]).
- **Non-smooth optimum** — the target SDF has Hessian singularities on $J_{\nabla\phi}$, so naively penalizing $\|D^2\phi\|^2$ everywhere is wrong.
- **Far-field decay matters.** Pure eikonal losses give little signal far from $\mathcal{S}$; methods like SIREN, HotSpot, HeatSDF address this differently (heat-equation logarithm, exponential growth penalty).

## Related

- [[Math - Ambrosio-Tortorelli Phase Field]] — used to mask higher-order eikonal regularizers on $J_{\nabla\phi}$.
- [[Graphics - Neural SDF Learning]] — the family of methods that solve eikonal problems with neural networks.
- [[Graphics - Medial Axis]] — the gradient jump set of the viscosity solution.
- [[Graphics - Ray Marching]] — the SDF that the eikonal equation defines is exactly what sphere tracing consumes.
