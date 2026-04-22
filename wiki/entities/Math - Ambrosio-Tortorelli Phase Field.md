---
title: "Ambrosio–Tortorelli Phase Field"
sources:
  - "[[raw/articles/Medial Axis Aware Learning of Signed Distance Functions]]"
tags:
  - phase-field
  - ambrosio-tortorelli
  - mumford-shah
  - variational-method
  - calculus-of-variations
created: 2026-04-22
updated: 2026-04-22
type: entity
---

A variational technique introduced by Ambrosio and Tortorelli (1992) for approximating free-discontinuity problems — most famously the **Mumford–Shah** functional from image segmentation. The trick: replace the explicit lower-dimensional jump set with a smooth auxiliary field $v\colon\Omega\to[0,1]$ that collapses to $0$ near the discontinuity and stays at $1$ everywhere else, in a transition band of width $\varepsilon$.

## The energy

For a small parameter $\varepsilon>0$, the canonical AT energy is
$$
\mathcal{L}_\text{AT}^\varepsilon[v] = \int_\Omega \varepsilon\|\nabla v\|^2 + \tfrac{1}{4\varepsilon}(v-1)^2 \,\mathrm{d}x.
$$

| Term | Effect |
|------|--------|
| $\varepsilon\|\nabla v\|^2$ | Smoothness; controls width of the transition band. |
| $\tfrac{1}{4\varepsilon}(v-1)^2$ | Pulls $v$ toward $1$ on sets of positive Lebesgue measure. |

Coupled with a "fidelity" term that wants $v$ to drop where some other field is irregular (image-intensity gradient in segmentation; SDF-Hessian in [[Graphics - Medial Axis Aware SDF Learning]]), the AT energy converges (as $\varepsilon\to 0$, in the sense of $\Gamma$-convergence) to the $(d-1)$-Hausdorff measure of the discontinuity set.

## Standard recovery construction

Around a $(d-1)$-rectifiable jump set $K$, set
$$
v^\varepsilon(x) = \begin{cases} 0 & \text{in a tube of radius } b_\varepsilon \text{ around } K, \\ 1 - \exp\!\big(\tfrac{b_\varepsilon - \mathrm{dist}(x,K)}{2\varepsilon}\big) & \text{else,} \end{cases}
$$
with $b_\varepsilon/\varepsilon\to 0$. This is the recipe used in App. A of [[Graphics - Medial Axis Aware SDF Learning]] to build a recovery sequence whose AT loss converges to $\mathcal{H}^{d-1}(J_{\nabla\phi})$.

## Why it's useful in machine learning

A jump set is a hard object to discretize directly: meshing it requires knowing it in advance, and adapting a neural network to predict it as a sharp set is awkward. The phase field $v$ is just a smooth function over $\Omega$ — easy to parameterize as a neural network. AT then turns "find the jump set and integrate over its complement" into "co-train an extra smooth network that effectively masks where regularizers should fire." The mask is differentiable and trains under standard gradient descent.

## Applications mentioned in the source

- **Image segmentation** (original Mumford–Shah motivation).
- **Brittle fracture** (Francfort–Marigo 1998, Bourdin et al. 2000); a recent deep-Ritz neural variant by Manav et al. 2024.
- **Medial-axis-aware neural SDFs** ([[Graphics - Medial Axis Aware SDF Learning]]) — the application introduced by this paper. Here AT approximates $\mathcal{H}^{d-1}(J_{\nabla\phi})$, and $v^2$ multiplicatively gates the higher-order term $\|D^2\phi\,\nabla\phi\|^2$ so it deactivates near the medial axis.

## Contrast with Modica–Mortola

The closely related **Modica–Mortola** functional uses a double-well potential and approximates a *two-phase* segmentation (like indoor/outdoor of a surface) — used by PHASE (Lipman 2021) for occupancy-style implicit representation. AT, by contrast, models a single field with a *thin discontinuity set*, which is what the medial-axis problem requires.

## Not quite the standard AT application

Classical AT in Mumford–Shah segments *image intensity*: the fidelity term $(\phi - \text{noisy input})^2$ + a Dirichlet term $\int(1-v)^2\|\nabla \phi\|^2$ whose jump set sits where the image has intensity discontinuities. [[Graphics - Medial Axis Aware SDF Learning]] replaces this with the SDF's second-order structural identity $D^2\phi\,\nabla\phi$: the gated term measures *gradient* discontinuity, not intensity discontinuity. Same topology (a phase field localises on a $(d-1)$-set), different geometric object being detected. This repurposing is what makes it the neural SDF contribution — not new AT theory, but a new target for an old tool.

The theoretical status is also weaker here than in classical Mumford–Shah: for image segmentation the full $\Gamma$-convergence was proved by Ambrosio–Tortorelli (1992); for the gradient-jump-set variant only the recovery sequence and existence of minimizers are established, matching the broader open $\Gamma$-limit problem for [[Math - Eikonal Equation|Aviles–Giga functionals]] in higher dimension.

## Related

- [[Math - Eikonal Equation]] — whose viscosity solutions have the jump set AT is being asked to model.
- [[Graphics - Medial Axis]] — the geometric object materialized by AT in the SDF setting.
