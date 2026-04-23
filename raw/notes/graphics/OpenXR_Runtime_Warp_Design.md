---
title: "OpenXR Runtime Warp Design — ATW / PTW / ASW"
date: 2026-04-24
type: note
tags:
  - graphics
  - xr
  - openxr
  - asw
  - atw
  - ptw
  - runtime
  - reprojection
  - motion-vectors
  - frame-generation
---

# OpenXR Runtime Warp Design — ATW / PTW / ASW

## Context

Follow-up to [`ASW_Mock_TechSharing.md`](ASW_Mock_TechSharing.md) and [`ASW_Mock_Sources_and_Evidence.md`](ASW_Mock_Sources_and_Evidence.md). The current project is implementing an OpenXR runtime on our own XR devices; ASW is one of the features alongside ATW and PTW. Three implementation-level questions surfaced while reading [[Graphics - Frame Generation for Real-Time Rendering]]:

1. Does interpolation-based frame generation (DLSS 3 FG style) work for ASW/ATW/PTW?
2. How are motion vectors generated when the previous display frame was a warp frame (no G-buffer)?
3. Should ATW, PTW, ASW be implemented as one combined pass, or staged in order?

This note captures the answers so we don't re-derive them later.

---

## 1. Interpolation does not work for ASW/ATW/PTW

Extrapolation is forced, not a design preference. Three tied reasons:

**Latency budget.** Interpolation holds the future frame back to display the intermediate first. At 90 Hz that's +11 ms on top of motion-to-photon, on top of an XR comfort ceiling that's already around 20 ms. ATW/PTW/ASW exist *to reduce* m-to-p latency; interpolation would undo the whole point. From [[Graphics - Frame Generation for Real-Time Rendering]] §Adjacent Topic:

> *FG interpolates between rendered frames at the same pose → higher perceived framerate. Space-Warp / late-warp extrapolate from one rendered frame to a fresher pose → lower perceived latency.*

**The future frame doesn't exist.** Interpolation needs both endpoints rendered. The runtime only has frame #N and a predicted head pose for time T+δ — not frame #N+1. Motion vectors describe `source → destination` (forward). Gather needs the inverse flow; there's no future frame to gather from. Same argument as [[XR - Space-Warp]] §Scatter-not-Gather.

**Different job.** FG raises perceived framerate at unchanged pose. ASW/PTW/ATW raise framerate *and* track the freshest pose — they compose with pose freshness, interpolation doesn't.

### Where interpolation FG is actually used

Not in XR. The shipping cases are:
- **Flat-screen desktop/console gaming** (DLSS 3 FG, AMD FSR 3 FG, Intel XeSS-FG). Seated player, no head tracking, +10 ms is annoying but not sickness.
- **Consumer TV motion smoothing** ("soap opera effect") — VFI without MVs, not engine-FG.

Cloud gaming is the interesting edge case: latency-sensitive but not head-tracked, so in principle could accept either mode. In practice cloud gaming chooses **extrapolation** (see [[Graphics - Post-Render Warp]]) because network RTT already consumed the latency budget.

---

## 2. Motion vectors come from the previous-frame transform, not the previous-frame image

Common misconception: MVs are an image-space delta between two rendered frames. They are not. MVs are computed during rasterization of the **current** frame, using per-vertex transform data from the **previous** frame. No prior color/depth texture is required.

### The rasterizer-pass recipe

When the app renders real frame #N, it already has per object:
- Current-frame transform: `MVP_current`
- Previous-frame transform: `MVP_prev` (snapshotted at last real render)

Vertex shader outputs both clip-space positions:

```glsl
out vec4 prev_clip;
void main() {
    vec4 curr = MVP_current * vertex_position;
    prev_clip = MVP_prev * vertex_position;  // SAME vertex, PREVIOUS transform
    gl_Position = curr;
}
```

Fragment shader writes the NDC delta into an MV render target:

```glsl
vec2 mv = (curr.xy / curr.w) - (prev_clip.xy / prev_clip.w);
```

Every modern engine has this as a standard pass — Unity URP's Motion Vector Pass, UE's `SceneVelocity` — and this is exactly the contract `XR_FB_space_warp` expects. Meta's Unity AppSW doc verbatim:

> *"We are writing the 3D motion vector data to the R, G, and B channels in the color texture."*
> *"the difference of the previous frame's vertex position transformed down to clip space"*

### Invariant for the runtime

**MVs need the previous *transform*, not the previous *image*.** Warp frames in between never produce G-buffers, but they don't break anything — the engine just keeps `MVP_prev` from the last real render. Skinned meshes need previous-frame skinning matrices; particles need previous positions; animated transforms need a "prev" snapshot every real render — all CPU-side bookkeeping, no dependency on whether the previous *display* frame was warped.

### What "previous" means under AppSW at half cadence

For 36 Hz app / 72 Hz display:

```
Display frame index:   0    1    2    3    4    5    6
Who renders:           app  warp app  warp app  warp app
Real frame index:      0         1         2         3
```

When the app renders real frame #1 (display frame #2), `MVP_prev` refers to real frame #0 (display frame #0) — **two display frame periods earlier**. So the MVs inside real frame #1 describe motion over two display-frame periods.

The runtime scales that MV for each warp target. Per Meta AppSW docs:

> *`motion_scale = (displayHz / appHz) − 1`*

For 72/36 = 2, scale = 1. For 90/45 or 120/60, also 1. For synthesizing a warp frame at display #3 from real frame #1, this means: apply the full observed MV to extrapolate forward one real-frame interval — equivalent to "continue object motion for one display frame."

The `motion *= 0.5` in the earlier ASW mock was for interpolating to the midpoint between two real frames, not for extrapolating forward by one warp frame — different target time, different scale. Parameterize the scale per target warp time; don't hard-code.

---

## 3. ATW + PTW + ASW: one kernel mathematically, two stages operationally

### Mathematical combination (one forward-scatter kernel)

Each of the three is a conditional contribution to the same per-pixel transform:

```
// ASW: per-pixel NDC offset from object motion
src_ndc = (x, y) + mv * motion_scale

// PTW: unproject with depth into source-camera 3D space
src_cam = unproject(src_ndc, d, P_src)

// PTW + ATW: apply camera pose delta source → target
// (rotation component alone = ATW; full 6-DOF = PTW)
// If depth unavailable, treat d → ∞ ⇒ pure ATW
tgt_cam = (Pose_target^-1 * Pose_source) * src_cam

// Reproject to target NDC
tgt_ndc = project(tgt_cam, P_tgt)
```

| Feature | Adds | Dropping it means |
|---|---|---|
| ASW | `+ mv * scale` | `mv = 0` → only camera motion, scene freezes |
| PTW | `unproject` with real `d` | `d = ∞` → all pixels at infinity, no parallax |
| ATW | rotation part of pose delta | identity → no head-pose correction |

Sequencing them into separate passes would be wasteful and worse quality:
- **Bandwidth**: each intermediate RT costs a read/write at framebuffer resolution per eye.
- **Resampling blur**: bilinear samples compound.
- **Disocclusion**: each pass opens new holes needing fill twice.

One forward-scatter kernel with atomic-max depth resolution + one hole-fill afterward. Matches AppSW and ExtraNet structurally and matches what the ASW mock already does for the ASW+PTW-with-depth case.

### Late ATW at scan-out — the mandatory second stage

Even with all three folded into one synthesis kernel, production XR runtimes run a **second, lighter reprojection at the compositor/scan-out stage**, and it's rotation-only (pure ATW).

Reason: **pose-sample cadence exceeds synthesis cadence.**

```
IMU sampling rate:     ~1000 Hz   (1 ms per sample)
Synthesis cadence:     72–90 Hz   (11–14 ms per frame)
Display scan-out:      same as synthesis rate
```

The synthesis pass picks a *predicted* display pose based on what it knew at kernel launch. 5–10 ms can pass between kernel launch and photon emission; the head keeps moving. Running the full ASW kernel again for the freshest sample is not feasible — too expensive, too close to scan-out. ATW is cheap (rotation-only, no depth, small-angle 2D UV offset) and safe to run just before the compositor hands pixels to the display.

Pipeline:

```
┌────────────┐    ┌────────────────────────┐    ┌──────────────────┐    ┌─────────┐
│ App submit │ →  │ Synthesis:             │ →  │ Late ATW at      │ →  │ Display │
│ color+d+MV │    │ ASW + PTW + ATW        │    │ scan-out         │    │ scan-out│
│ + pose_src │    │ (one scatter pass)     │    │ (rotation only,  │    │         │
└────────────┘    │ pose_target_predicted  │    │  latest pose)    │    │         │
                  └────────────────────────┘    └──────────────────┘    └─────────┘
     app rate           synthesis rate                scan-out rate
```

The late ATW re-uses the ATW math with `Pose_target` updated and `d = ∞`. It doesn't disturb object-motion or parallax corrections already baked into the intermediate — a 2D rotation in screen space approximately preserves them for small Δangle.

### Implementation implications for our runtime

1. **One synthesis kernel, conditional inputs.** Implement ASW/PTW/ATW as conditional contributions in one forward-scatter compute shader. If app didn't submit MVs → `mv = 0` ⇒ pure PTW. If depth not bound → `d` = large constant ⇒ pure ATW. Don't build three kernels; don't branch per feature.

2. **Keep the late ATW as a separate pipeline stage.** Different location (scan-out, not synthesis), different input (latest IMU sample), different constraints (no atomic ops or hole-fill affordable), different shader (screen-space 2D rotation per eye). Running ATW twice — once inside synthesis with predicted pose, once at scan-out with fresh pose — is intentional: first contributes to synthesis re-scaling, second hides remaining m-to-p latency.

3. **Compose pose deltas, not warps.** When the synthesis kernel runs, pre-multiply `Pose_target_predicted^-1 · Pose_source` into one 4×4 that the shader applies once. Don't do source → intermediate → target in two shader multiplies.

4. **ASW ⊇ PTW ⊇ ATW in the input contract.** A runtime that accepts `XR_FB_space_warp` (color + depth + MV) also implicitly handles the PTW and ATW cases just by varying inputs. A runtime that only does ATW can ignore depth and MV channels even if submitted. Design the input struct and kernel so the feature degrades cleanly when inputs are absent.

---

## 4. Pipeline stages — app submission (1st) and runtime compositor (2nd)

XR rendering splits cleanly into two stages that run at decoupled cadences.

| Stage | Aka | Owner | Rate | Produces |
|---|---|---|---|---|
| **1st** | "application submission" | App / Unity / UE | App rate (36 / 45 / 72 Hz) | Color + depth + MVs per eye, written into OpenXR swapchain images |
| **2nd** | "runtime compositor" | Runtime | Display rate (72 / 90 / 120 Hz) | Composited, warped, lens-corrected, scanned-out image |

Calling the 2nd stage "rendering" is loose shorthand — no geometry is rasterized there. It's image-space post-processing of the app's submitted frame. OpenXR spec calls the two stages **application** and **compositor**.

### What the 2nd stage actually does

```
App submits swapchain (color + depth + MV) per eye per layer
│
├─ Layer composition    ← merge projection + quad + cylinder layers (UI, overlays)
│
├─ Warp pass            ← ASW + PTW + ATW in one forward-scatter kernel
│    (synthesis)           (the core warp — §3 above)
│
├─ Late ATW             ← rotation-only fix-up, latest IMU sample, just before scan-out
│
├─ Lens distortion      ← pre-distort so physical optics produce rectilinear output
│
├─ Chromatic aberration ← per-channel UV offset to compensate lens CA
│
├─ Color / tone         ← device-specific gamut
│
└─ Scan-out             ← DMA to display
```

Everything below "layer composition" is runtime-side; the app never touches it. Our warp work lives in *Warp pass (synthesis)* and *Late ATW*, distinct from the lens/CA/scan-out stages which always run and are unrelated to head-pose prediction.

### Cadence decoupling is the whole point

- App renders at what it can afford (often half display rate with ASW enabled).
- Runtime composites at display rate, regardless of whether a fresh app submission arrived this cycle.
- If no fresh app frame arrived, runtime re-uses the previous submission and re-warps for the new pose. This is why ASW can hold 72/90/120 Hz display refresh while the app renders at 36/45/60 Hz.

### Freshness chain

The app's 1st stage is not pose-free: it queries `xrLocateViews` at render-start and rasterizes against a predicted pose. The 2nd stage corrects the delta between that prediction and the actual display pose. Three pose samples are in play, at increasing freshness:

```
App's prediction (stalest)  →  Warp's prediction (fresher)  →  Late-ATW's sample (freshest)
```

Each stage only corrects the delta the prior stage couldn't see.

---

## 5. Where the runtime code lives — in-process vs out-of-process

Orthogonal architectural decision to the 1st/2nd-stage split: does the runtime's compositor code run *inside* the application's process, or as a separate system service?

| | In-process | Out-of-process |
|---|---|---|
| Runtime runs as | DLL loaded into app's address space | Separate system service / daemon |
| Communication | Direct function calls + shared memory within process | IPC: shared mem + sockets / pipes / D-Bus |
| GPU context | Same as app | Separate; swapchain textures shared via GPU handles |
| Lifetime | Dies with the app | Persistent; serves many apps across launches |
| Crash blast radius | Runtime bug → app crash; app crash → runtime gone | Either can crash independently |

### The OpenXR loader is always in-process

The loader (`openxr_loader.dll/.so`) is always in-process — it's what the app links against. The question is whether the runtime's *work* happens in that same DLL or ships across a process boundary.

```
┌─ Application process ─────────────────────────────────┐
│                                                        │
│  App → openxr_loader → API layers → runtime client DLL │
│                                              │         │
└──────────────────────────────────────────────│─────────┘
                                               │  IPC if out-of-process
                                               ▼
                                ┌─ Runtime service process ─┐
                                │  Compositor (ASW/PTW/ATW) │
                                │  Sensor fusion / tracking │
                                │  Display driver bridge    │
                                │  Guardian / boundary      │
                                │  System UI / overlays     │
                                └───────────────────────────┘
```

### Production runtimes are almost always out-of-process

SteamVR, Meta Quest OS, Oculus PC runtime, Microsoft WMR, Monado (service mode) — all ship as system services with a thin in-process client shim. Reasons:

1. **Exclusive hardware ownership.** Only one process owns the HMD display, IMU stream, and compositor timeline. Overlays, dashboards, system UI need a shared arbiter that outlives any one app.
2. **Session continuity.** Tracking, lens calibration, guardian bounds persist across app launches — the runtime keeps running between apps.
3. **Overlay / system UI.** The runtime renders the OS home, notifications, recenter UI; those composite *with* the app's output, which requires the compositor to be outside the app.
4. **Crash isolation.** App GPF shouldn't kill the HMD session; a runtime bug shouldn't crash every XR app on the system.
5. **Privilege boundary.** Kernel-driver access (direct-mode display, HMD USB) centralizes cleanly in one privileged service.

### When in-process is used

- **Development / test runtimes** — simpler, no IPC plumbing.
- **Embedded single-app devices** — a standalone HMD running one hermetic app at a time doesn't need multi-client arbitration.
- **Hot-path calls inside production runtimes.** `xrLocateSpace` / `xrLocateViews` get called many times per frame; IPC would be prohibitive. Served from an in-process shared-memory pose cache that the service refreshes periodically.

### The realistic hybrid for our runtime

**In-process (thin client DLL):**
- OpenXR API surface (object creation, extension dispatch, validation).
- Pose query hot paths served from shared-memory pose cache.
- Swapchain image acquisition (shared-handle to the service's texture).
- Event queue pumping.

**Out-of-process (runtime service):**
- Compositor — where the ASW/PTW/ATW warp from §3 lives.
- Sensor fusion / tracking (IMU + cameras → pose stream).
- Display driver interface (scan-out timing, VSync).
- Guardian / boundary / room setup.
- Controller input multiplexing.
- System UI rendering.

**IPC mechanisms needed:**
- **Shared-memory ring buffer** for pose/controller state — service writes, client reads lock-free. Avoids IPC on `xrLocateSpace`.
- **Shared GPU textures** for swapchain images — DXGI shared handles on Windows, DMA-BUF on Linux, Android HardwareBuffer for standalone.
- **Timeline semaphores / fences** for cross-process GPU sync — service must know when the client's render is done before composing.
- **Named pipe / socket** for control-plane RPC (session begin, swapchain create, layer submit metadata, event push).

### Implications for the warp work

1. **Warp runs in the service's GPU context**, not the app's. Client DLL never sees the ASW shader code.
2. **Layer composition happens in the service.** Client submits layer metadata + texture handles; service composites.
3. **A single-process debug mode is still worth building.** Link service logic directly into the client DLL behind a flag, skip IPC. Invaluable for testing warp kernels without fighting shared-handle bugs.
4. **When prior sections say "runtime does the 2nd rendering" — that "runtime" is overwhelmingly the out-of-process service.** The warp pass runs in its GPU context at display cadence, reading from swapchain textures the app populated.

---

## 6. Runtime-level post-processing — what belongs here, what doesn't

The 2nd-stage compositor is already doing post-processing (lens distortion, chromatic aberration, tone mapping). Question surfaced whether to add anti-aliasing there too. Answer: **AA belongs in the engine, not the runtime.** Some post-processing is legitimately runtime-side; AA mostly isn't.

### Why AA at the runtime is the wrong layer

AA flavors have natural homes, and the runtime owns only the weakest category:

| AA type | Must run at | Why |
|---|---|---|
| **MSAA** | Rasterizer (engine) | Needs sub-sample coverage at triangle setup; lost by compositor stage. |
| **SSAA / supersampling** | Engine renders high, compositor downsamples | Extra pixels must come from geometry; compositor can only downsample. |
| **TAA / DLAA / DLSS / FSR2** | Engine | Needs MVs, jittered projection, history buffer, per-frame seed. |
| **FXAA / SMAA (spatial post)** | Anywhere 2D | Runtime *could* do it — but shouldn't. |

Even the one case where a runtime *could* run AA (FXAA/SMAA) is a bad fit:

1. **Pipeline order has no good slot.** Before warp → warp resample undoes edge work. After warp, before lens distortion → lens resample re-aliases. After lens distortion → edges are barrel-distorted; heuristics don't transfer, and budget to scan-out is ~1 ms.
2. **Lens distortion dominates final-image aliasing.** Any AA effort that doesn't target that specific pass is fighting the wrong enemy.
3. **Double-processing.** Apps already run TAA/FXAA/SMAA in-engine before submitting. Runtime AA re-blurs what's been resolved, or fights what the app is about to blur. Apps expect submitted pixels to reach the display.
4. **Foveation breaks naive post-process.** Foveated swapchain textures (variable shading rate / variable resolution) need foveation-aware passes; position-independent AA blurs across tile seams. Requires engine cooperation, defeating the point of runtime-side.
5. **Quality ceiling is low.** Post-process spatial AA on an already-composited frame is strictly inferior to engine-side TAA. Wrong layer for the problem.

### What production runtimes actually ship

- **Meta Quest compositor**: lens distortion + CA + optional sharpening. No dedicated AA pass.
- **SteamVR**: same pattern; optional post-lens sharpening; supersampling as a per-app RT scale knob (SSAA via engine cooperation), not a compositor AA pass.

### What runtime-level post-processing *is* legitimate

These either must run at the runtime or only make sense there:

1. **Lens distortion** — mandatory. Physical optics dictate the precompensation mesh.
2. **Chromatic aberration correction** — mandatory. Per-channel UV offset, lens-specific.
3. **Sharpening (CAS / RCAS) after lens distortion** — counteracts softening from the barrel-precomp resample. Cheap, localized, real quality win. *The one case where runtime post genuinely pays off.*
4. **Gamut / tone mapping for device output** — HDR pipelines, panel-specific correction.
5. **Supersampling support** — expose a runtime resolution-scale knob; apps create larger swapchains; compositor downsamples. Opt-in via app cooperation, effectively SSAA. Closest thing to "runtime AA" that works.

### Decision for our runtime

**Ship in the compositor:**
- Lens distortion + CA (mandatory for any XR runtime).
- Post-lens sharpening (CAS-style). Small, real quality improvement, pays for itself.
- A runtime-exposed resolution-scale knob for supersampling (apps render at 1.2× / 1.5× display res into larger swapchains; compositor downsamples). This is the only lever that actually improves AA without fighting the app.

**Don't ship in the compositor:**
- Spatial AA (FXAA / SMAA). Strictly worse than what apps do, costs frame budget, breaks with foveation. If apps have weak AA, the fix is documentation/requirements, not second-guessing at the compositor.
- TAA / DLAA / DLSS-class temporal AA. Requires MVs + jittered projection + history; engine already has the right context.

### The underlying principle

Post-processing splits by what context the stage has access to:

| Layer | Has access to | Good at |
|---|---|---|
| **Engine (1st stage)** | G-buffer, per-frame data, scene | TAA, bloom, motion blur, DOF, color grading, HDR transforms before tonemap |
| **Runtime (2nd stage)** | Submitted 2D image + OpenXR metadata + pose stream | Lens distortion, CA, sharpening, tone/gamut, warp, supersample downsample |

Anything needing scene context belongs in the 1st stage. Anything device-specific or pose-dependent belongs in the 2nd. **AA sits firmly in the first category** — with the narrow supersampling exception where the extra pixels come from the engine and the runtime only does the downsample.

---

## Open questions

- **Exact IMU-to-scan-out latency budget on our target device.** Determines how much head motion the late ATW is covering, which bounds the acceptable small-angle rotation approximation error.
- **Whether to expose a runtime-internal "warp frame at display sub-rate" path.** Some platforms warp twice between real renders (e.g., 90/30). If so, the MV scale parameterization needs to handle multiple target times per real frame, not just one.
- **Composition with foveated rendering.** Fixed / eye-tracked foveation changes effective resolution per tile; the warp kernel's atomic-max resolution needs to respect the tile layout or you get seams.
- **Target platform stack (Windows / Linux / Android standalone).** Drives shared-texture mechanism (DXGI shared handles vs DMA-BUF vs AHardwareBuffer) and cross-process sync primitives (D3D12 fences vs Vulkan timeline semaphores).
- **Loader strategy.** Use Khronos' reference OpenXR loader and ship only a runtime manifest + client DLL, or fork/replace the loader. The reference loader covers validation layers and extension dispatch for free.
- **Service activation model.** Autostart daemon at boot vs on-demand launch when the first OpenXR app starts. Trade-off: cold-start latency on first app launch vs always-on memory/power cost.
- **Hot-path vs round-trip boundary.** Exact list of OpenXR entry points served locally from cached state vs. IPC-round-tripped. `xrLocateSpace` local; `xrEndFrame` round-trips. Borderline cases (swapchain image acquire timing, event poll) need to be decided up front.
- **Sharpening kernel choice.** CAS (AMD) vs RCAS vs a simpler unsharp mask for the post-lens sharpening pass. Quality-vs-cost trade-off is panel- and lens-dependent; needs measurement on our hardware.
- **Runtime resolution-scale contract.** How apps discover and select supersampling scale — via a vendor OpenXR extension, session config, or per-app override. Also whether the scale is uniform or per-eye / per-layer.

---

## References

- Meta, *Application SpaceWarp for Unity*, `developers.meta.com/horizon/documentation/unity/unity-asw/` — motion-vector generation recipe and `motion_scale = displayHz/appHz − 1`.
- Meta, *Application SpaceWarp — How it Works*, `developers.meta.com/horizon/documentation/unity/os-app-spacewarp/` — forward-scatter semantics.
- Khronos, `XR_FB_space_warp` OpenXR extension — standardized app→runtime input contract for color + depth + MV.
- Mark, McMillan, Bishop, *Post-Rendering 3D Warping*, I3D '97 — canonical depth-image forward warp.
- [[XR - Space-Warp]] — entity page covering scatter-not-gather and lineage.
- [[Graphics - Frame Generation for Real-Time Rendering]] — topic page covering interpolation-vs-extrapolation trade-offs and depth-aware forward splat primitive.
