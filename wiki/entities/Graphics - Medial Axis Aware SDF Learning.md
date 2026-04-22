---
title: "Medial Axis Aware Learning of Signed Distance Functions"
sources:
  - "[[raw/articles/Medial Axis Aware Learning of Signed Distance Functions]]"
tags:
  - neural-sdf
  - medial-axis
  - phase-field
  - ambrosio-tortorelli
  - eikonal-equation
created: 2026-04-22
updated: 2026-04-22
type: entity
---

A variational neural method (Weidemaier, Norden-Smoch, Rumpf, U. Bonn, 2026) that jointly learns a signed distance function and an explicit phase-field approximation of its [[Graphics - Medial Axis|medial axis]] from an unoriented point cloud. Two networks $\phi_\theta$ (SDF) and $v_\eta$ (phase field) are optimized simultaneously under a higher-order [[Math - Ambrosio-Tortorelli Phase Field|Ambrosio–Tortorelli]] loss, producing both detailed surface reconstruction and uniformly accurate distances in the far field.

## Core idea

The SDF is the viscosity solution of the [[Math - Eikonal Equation|eikonal equation]] $\|\nabla\phi\|=1$, and its gradient *must* be discontinuous on the medial axis. Any global higher-order regularizer therefore has to switch off there. The paper does this by learning a phase field $v\in[0,1]$ that collapses to $\approx 0$ in an $\varepsilon$-neighborhood of $J_{\nabla\phi}$ and gates the regularizer:

$$
\mathcal{L}_\text{HO}^\varepsilon[\phi,v] = \int_\Omega v^2\,\|D^2\phi\,\nabla\phi\|^2 + \varepsilon^2\|D^2\phi\|^2\,\mathrm{d}x
$$

The factor $D^2\phi\,\nabla\phi$ encodes the SDF's characteristic linear growth in the gradient direction (it vanishes wherever $\phi$ is twice differentiable). The $\varepsilon^2\|D^2\phi\|^2$ term is an isotropic well-posedness regularizer.

## Total loss

$$
\mathcal{L}_\text{total}^\varepsilon[\phi,v] = \gamma_\text{HO}\mathcal{L}_\text{HO}^\varepsilon + \gamma_\text{AT}\mathcal{L}_\text{AT}^\varepsilon[v] + \gamma_\text{recon}\mathcal{L}_\text{recon}^\varepsilon[\phi] + \gamma_\text{eik}\mathcal{L}_\text{eik}^\varepsilon[\phi] + \gamma_\text{exp}\mathcal{L}_\text{exp}[\phi]
$$

The HO+AT pairing is the neural realization of a functional proposed by **Aviles and Giga (1987)** in the context of smectic liquid crystals: a Modica–Mortola-type energy with higher-order derivatives whose sharp-interface limit depends on the jump set of the gradient of an a.e. eikonal solution. The full $\Gamma$-convergence theory is only partially resolved — Ambrosio–De Lellis–Mantegazza (1999) in 2D, Jin–Kohn (2000) for the lower bound, De Lellis (2002) showing higher-dimensional behavior differs — which is why this paper's theorem stops at existence and a recovery sequence, without a matching $\liminf$.

| Term | Form | Purpose |
|------|------|---------|
| $\mathcal{L}_\text{eik}^\varepsilon$ | $\tfrac{1}{\varepsilon}\int(\|\nabla\phi\|^2-1)^2$ | Eikonal fidelity |
| $\mathcal{L}_\text{HO}^\varepsilon$ | see above | Linear-growth structure away from $J_{\nabla\phi}$ |
| $\mathcal{L}_\text{AT}^\varepsilon$ | $\int \varepsilon\|\nabla v\|^2 + \tfrac{1}{4\varepsilon}(v-1)^2$ | $(d-1)$-Hausdorff measure of the jump set |
| $\mathcal{L}_\text{recon}^\varepsilon$ | $\tfrac{1}{\varepsilon^2}\int_\mathcal{S}\phi^2$ | Surface anchoring on the input point cloud |
| $\mathcal{L}_\text{exp}$ | $\sum_p\int e^{-\alpha_p|\phi|^p}$ | Bias toward viscosity (maximal $|\phi|$); $\alpha=(100,100,10)$ |

**Why $\mathcal{L}_\text{exp}$ is not redundant with AT.** Figure 4 gives an a.e. eikonal solution whose jump set is *shorter* than the viscosity solution's. So minimizing $\mathcal{H}^{d-1}(J_{\nabla\phi})$ alone can pick a non-viscosity solution. The exponential penalty breaks the tie by preferring maximal $|\phi|$. There is no proof that the combination selects viscosity — only empirical evidence.

**Signed-ness is free.** No inside/outside segmentation or oriented normals are required. If the optimizer slid to the *unsigned* distance, its gradient would jump on $\mathcal{S}$ itself (a $(d-1)$-set), inflating $\mathcal{H}^{d-1}(J_{\nabla\phi})$; AT penalizes exactly this, so the signed solution is energetically favoured (up to a global sign).

## Networks

- **SDF $\phi_\theta$:** QuaNet (quadratic neural network, Fan et al. 2020) — MLP with 256 hidden units, quadratic pre-activation expansion + SIREN activations. 4 layers in 2D, 8 in 3D.
- **Phase field $v_\eta$:** ResNet with SIREN activations. 4 blocks × 64 (2D) / 6 blocks × 128 (3D). Output sigmoid-clamped to $[0,1]$.
- Optimizer: Adam, separate LRs ($5\times10^{-5}$ SDF, $5\times10^{-4}$ PF), $\beta_2$ also differs (0.98 vs 0.999).
- $\varepsilon=10^{-3}$ (2D), $10^{-4}$ (3D); domain $\Omega=[-1.2,1.2]^d$.

## Three-phase training schedule

1. **SDF init (epochs 0–5):** $\phi_\theta$ initialized to the unit-sphere SDF $\|x\|-1$. $v\equiv 1$ frozen. Large $\gamma_\text{exp}$ to grow the field outward; low $\gamma_\text{eik}, \gamma_\text{HO}$ to suppress ghost geometry.
2. **Joint optimization (epochs 5–20):** Activate phase field. Ramp $\gamma_\text{eik}, \gamma_\text{HO}$ up to target; ramp $\gamma_\text{exp}$ down.
3. **SDF refinement (epochs 20–30):** Freeze $v_\eta$. Continue training $\phi_\theta$; additionally evaluate $v_\eta^2\|D^2\phi_\theta\nabla\phi_\theta\|^2$ on the input point cloud.

3D weight schedule (App. C):
- Phase 1: $(\gamma_\text{HO},\gamma_\text{AT},\gamma_\text{recon},\gamma_\text{eik},\gamma_\text{exp}) = (1, 0.02, 0.01, 0.05, 500)$
- Final: $(2.5, 0.2, 0.5, 0.2, 200)$

## Adaptive sampling (3D)

Monte Carlo quadrature with local grid refinement: cells where $|\phi_\theta|<2^{-i}\tau_\text{sdf}$ or $v_\eta<\tau_\text{pf}$ are subdivided ($\tau_\text{sdf}=0.1$, $\tau_\text{pf}=0.75$). Refined cells get sample weights $(2^{id}n)^{-1}h^d$. Concentrates samples near the surface and the predicted medial axis. Surface reconstruction loss uses HeatSDF's local weighting scheme.

## Theory (Theorem 3.1 + Remark 3.1)

- **Existence:** For fixed $\varepsilon>0$, a minimizing tuple $(\phi^\varepsilon, v^\varepsilon)\in H^2(\Omega)\times H^1(\Omega)$ exists. Proof (App. A): boundedness from finite total loss + Poincaré-type estimate; weak compactness in $H^2\times H^1$; lower semi-continuity for each term, with the bilinear $v\,D^2\phi\,\nabla\phi$ term handled via $L^6$ Sobolev embedding for $d\in\{2,3\}$ giving weak convergence in $L^{6/5}$ and a.e. identification of the limit.
- **Recovery sequence (limsup):** A mollification of $\phi$ on a tubular neighborhood of $J_{\nabla\phi}$ of radius $b_\varepsilon$ with $b_\varepsilon/\varepsilon\to 0$ and $\varepsilon^2/b_\varepsilon\to 0$, paired with $v^\varepsilon(x)=1-\exp((b_\varepsilon-\mathrm{dist}(x,J_{\nabla\phi}))/2\varepsilon)$ outside the band, recovers all loss components in the limit. The matching $\liminf$ inequality (and full $\Gamma$-convergence) is left open.

## Results

**SRB (Surface Reconstruction Benchmark, 5 shapes):**

| Method | $d_C$ | $d_H$ | $E_n$ | $E_\text{eik}^\Omega$ | $E_\text{SDF}^\mathcal{N}$ | Time |
|--------|-------|-------|-------|----------------------|----------------------------|------|
| GSD (grid) | 0.0302 | 0.1171 | 0.1249 | 0.0375 | 0.0772 | 11 min |
| 1-Lip | 0.0573 | 0.2468 | 0.1188 | 0.0429 | 0.0917 | 74 min |
| HeatSDF | 0.0147 | 0.1521 | 0.0378 | 0.2228 | 0.0134 | 32 min |
| Hessian | 0.0102 | **0.0521** | 0.0116 | 0.7693 | 0.0283 | 19 min |
| HotSpot | 0.0105 | 0.0704 | 0.0114 | 0.0905 | 0.0712 | 6 min |
| **Ours** | 0.0104 | 0.0645 | **0.0110** | **0.0106** | **0.0103** | 11 min |

Decisive lead on global SDF accuracy ($E_\text{eik}^\Omega$, $E_\text{SDF}^\mathcal{N}$), tied for best on near-field surface reconstruction ($d_C$, $E_n$), at competitive runtime — despite using two networks and second derivatives. ~85³ DoF total (vs GSD's 128³).

**Thingi10k (40 shapes, 20 random + 20 tagged `sculpture`/`scan`):**

| Method | $d_C$ | $d_H$ | $E_n$ | $E_\text{eik}^\Omega$ | $E_\text{eik}^\mathcal{N}$ | $E_\text{SDF}^\Omega$ | $E_\text{SDF}^\mathcal{N}$ |
|--------|-------|-------|-------|-----------------------|-----------------------------|-----------------------|-----------------------------|
| 1-Lip† | 0.0536 | 0.1576 | 0.1413 | 0.0728 | 0.1109 | 0.8607 | 0.1109 |
| HeatSDF† | 0.0448 | 0.1689 | 0.0893 | 0.2136 | 0.1588 | 0.3215 | 0.0172 |
| Hessian | **0.0110** | **0.0472** | 0.0753 | 0.7615 | 0.8185 | 0.4841 | 0.0318 |
| HotSpot | 0.0130 | 0.0566 | 0.0818 | 0.1069 | 0.2143 | 0.4266 | 0.0671 |
| **Ours** | 0.0124 | 0.0847 | **0.0741** | **0.0769** | **0.0189** | **0.0253** | 0.0871 |

† 1-Lip and HeatSDF failed to converge on some shapes and were excluded from their averages on those — the reported numbers flatter the two methods. Ours is the only method reporting on the full 40.

Dominates all four distance-fidelity metrics by a wide margin (e.g. $E_\text{eik}^\Omega$: ours 0.077 vs next-best 0.107; $E_\text{SDF}^\Omega$: 0.025 vs next-best 0.32). Second on Chamfer/Hausdorff behind Hessian, whose global SDF error is catastrophic. This is the robustness story — competitors that win near-field metrics either collapse on global metrics or fail outright on harder topology.

**Sphere tracing:** Demonstrated on torus and hand shapes — fewer iterations per pixel than Hessian/HotSpot at similar visual quality, the direct payoff of accurate far-field SDF.

**Ablation (§5.2.3):** The eikonal + HO combination *without* the phase field fails to recover high-quality SDFs. The phase field is load-bearing, not cosmetic — without it, the HO term collides with the unavoidable Hessian singularity on the medial axis and corrupts the global solution.

## Limitations

- **Co-dimension-2 medial-axis components** (e.g., the rotation axis of a torus) are not penalized by the AT energy — they show up as artifacts in $v_\eta$.
- **Complex topology** with the fixed parameter set: the phase field fails to capture intricate medial axes; the paper notes this would need smaller $\varepsilon$, larger $\alpha_p$, and bigger networks but currently does not converge.
- **No analytical guarantee** that $\mathcal{L}_\text{exp}$ actually selects the viscosity solution among a.e. eikonal solutions.

## Plug-in usability

As a proof of concept the authors drop the HO + AT regularizer into HeatSDF's second step:

$$\mathcal{L}_\text{Modified} = \mathcal{L}_\text{HeatSDF}^{2\text{nd}} + \mathcal{L}_\text{HO}^\varepsilon + \mathcal{L}_\text{AT}^\varepsilon$$

This yields visibly sharper level sets near the surface and reduced surface noise in renderings — suggesting the medial-axis-aware regularizer is a generic add-on for other neural SDF pipelines, not just standalone.

## Comparison context

Sits within the [[Graphics - Neural SDF Learning|neural SDF lineage]]: SIREN (eikonal-only) → DiGS (Laplacian L²) → StEik / Hessian / Neural-Singular-Hessian (near-field Hessian degeneracy) → HotSpot (heat–SDF coupling, one-step) → HeatSDF (Crane-style two-step neural variant) → 1-Lip (Lipschitz constraint) → **this work** (global second-order + learned medial-axis mask).

Distinct from PHASE (Lipman 2021), which uses Modica–Mortola two-phase segmentation for the *surface* and post-processes for an SDF; here AT is used to model the *gradient* jump set, with $\phi$ directly representing the SDF.
