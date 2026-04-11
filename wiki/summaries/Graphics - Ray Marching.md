---
title: Ray Marching — Summary
sources:
  - "[[Ray Marching]]"
tags:
  - real-time-rendering
  - ray-marching
  - computer-graphics
  - GLSL
  - signed-distance-functions
created: 2026-04-11
updated: 2026-04-11
type: summary
recommend: partial
distil_times: 0
---

# Ray Marching — Michael Walczyk

## Summary

A beginner-friendly tutorial on ray marching (sphere tracing) in GLSL. Covers signed distance functions (SDFs), the ray marching loop with distance-aided stepping, ray generation from a virtual camera, normal estimation via gradient computation, basic diffuse lighting, and SDF distortion for procedural shapes. Uses TouchDesigner but is portable to any GLSL environment.

## Key Points

- **Signed Distance Functions (SDFs):** mathematical functions that return the distance from a point to the nearest surface. "Signed" because negative = inside, zero = on surface, positive = outside.
- **Sphere tracing:** at each step along a ray, evaluate the SDF and advance by that distance — guaranteed not to overshoot. Much more efficient than fixed-step marching.
- **Two early-exit optimizations:** break when distance < 0.001 (hit) or total distance > 1000.0 (miss).
- **Ray generation:** each fragment shader pixel generates a unique ray from camera through an image plane, using remapped UV coordinates [-1, 1].
- **Normal estimation:** nudge point p along each axis by a small epsilon, evaluate SDF at both offsets, compute the gradient — works for arbitrarily complex shapes.
- **Diffuse lighting:** standard Lambertian dot(normal, light_direction).
- **SDF distortion:** adding sinusoidal displacement to an SDF creates organic procedural shapes with correct normals automatically.

## Entities Mentioned

- Michael Walczyk — author
- Inigo Quilez — referenced as SDF resource
- TouchDesigner — rendering environment used
- Shadertoy — referenced as ray marching showcase

## Related

- Potential topics: SDF primitives, sphere tracing, procedural geometry, fragment shaders

## Recommend Reason

**Partial.** This is a solid introductory tutorial — well-structured with clear progression from SDFs to complete shading. However, it stays at a beginner level and covers only one primitive (sphere), basic Lambertian lighting, and a single distortion technique. If you're new to ray marching, it's a good foundation worth distilling for the core concepts (SDF, sphere tracing, gradient-based normals). If you already know the basics, the article won't offer much new — Inigo Quilez's site would be more valuable at that point.
