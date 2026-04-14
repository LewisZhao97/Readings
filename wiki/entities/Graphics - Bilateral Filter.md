---
title: "Bilateral Filter"
sources:
  - "[[raw/articles/Image Guided Depth Super-Resolution for Spacewarp in XR Applications]]"
  - "[[raw/articles/Space Warp with Depth Propagation in XR Applications]]"
tags: [image-processing, bilateral-filter, edge-preserving, depth-map]
created: 2026-04-14
updated: 2026-04-14
type: entity
---

# Bilateral Filter

Edge-preserving smoothing filter (Tomasi & Manduchi 1998; Durand & Dorsey 2002). Unlike a Gaussian blur, which averages purely by spatial proximity and smears edges, the bilateral filter weights each neighbor by **both** spatial proximity and **intensity similarity**, so it averages only within regions of similar value and leaves discontinuities intact.

## Form

$$
BF(I_p) = \frac{1}{W_p} \sum_{q \in N} G_{\sigma_s}(\|p - q\|)\; G_{\sigma_r}(I_p - I_q)\; I_q
$$

- $G_\sigma$ — Gaussian kernel with scale $\sigma$
- $\sigma_s$ — spatial extent
- $\sigma_r$ — range (intensity) extent
- $W_p$ — normalizer (sum of weights)

## Joint / Cross Bilateral Filter

When the range weight is computed on a **guide** signal different from the one being filtered, the filter is called *joint* or *cross* bilateral. Common use: upsample a low-resolution signal using a high-resolution guide (e.g., depth map guided by its co-registered color image). This is the foundation of both Samsung Space-Warp papers — see [[XR - Depth Reconstruction for Space-Warp]].

## Appearances in This Wiki

- **Depth super-resolution for XR** (Peri & Xiong 2021): three-weight variant with spatial, color-intensity, and pose-derived depth terms. See [[XR - Depth Super-Resolution vs Depth Propagation]].
- **Depth propagation for XR** (Xiong & Peri 2021): the smooth term's per-edge weight $W_{pq}$ is bilateral-flavored (color-image + original-depth-map similarity), embedded inside a global optimization rather than applied as a one-shot filter.

## Properties

- Edge-preserving: discontinuities survive because $G_{\sigma_r}$ kills cross-edge weights.
- Tunable: $\sigma_s$ controls smoothing radius; $\sigma_r$ controls edge sensitivity. Larger $\sigma_r$ → closer to plain Gaussian.
- Non-linear (weights depend on signal), so not separable in general; efficient approximations exist.
