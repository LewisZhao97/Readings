---
title: "Mocking Application SpaceWarp (ASW) in Unity URP — Technical Deep Dive"
date: 2026-04-16
type: note
tags:
  - graphics
  - xr
  - asw
  - space-warp
  - unity
  - urp
  - frame-extrapolation
  - motion-vectors
---
# Mocking Application SpaceWarp (ASW) in Unity URP — Technical Deep Dive

A proof-of-concept that reproduces Oculus/Meta's **Application SpaceWarp (ASW)** behaviour inside a plain Unity URP project — no XR runtime, no SDK. The goal is to run the full algorithm end-to-end on desktop, validate correctness visually, and land on an implementation structure that is portable to a real XR compositor layer later.

---

## 1. What ASW actually is

ASW is a **frame-extrapolation** technique used by XR runtimes to halve the application's shading cost:

- The application renders at **half display rate** (e.g. 45 Hz when the HMD is 90 Hz).
- Between each "real" frame, the compositor **extrapolates** a synthetic intermediate frame by **forward-warping** the previous frame using its per-pixel motion vectors and depth.
- The user perceives a full-rate image; the GPU only shades every other frame.

Two data streams the application must deliver to the runtime:

1. **Color** — the final post-processed image.
2. **Motion Vectors** — screen-space 2D displacement of each pixel from previous frame to current frame (combines camera + per-object motion). URP already produces these.
3. **Depth** — for occlusion resolution during scatter.

The runtime does *not* re-render the scene on warp frames. It reuses the last real frame's textures and reprojects them.

---

## 2. What we built

A self-contained URP implementation that mocks this on a single `Main Camera`:

- **Even frames** (`Time.frameCount % 2 == 0`) → real rendering. The scene is drawn normally and its color / depth / motion are **captured** into history RTs.
- **Odd frames** → real rendering is **skipped entirely**. A compute-shader warp pass reads the captured history and synthesises a new frame.

Three source files carry the whole feature:

| File | Role |
| --- | --- |
| `Assets/Scripts/ASWDualRendererFeature.cs` | URP `ScriptableRendererFeature` + per-frame coordinator |
| `Assets/Shaders/ASWWarp.compute` | 3-kernel compute shader: clear → scatter → resolve |
| `Assets/Shaders/ASWHistoryCapture.shader` | MRT blit shader that copies color / depth / motion into the three history RTs in a single draw |

---

## 3. Architecture: dual-renderer strategy

### The problem with a single renderer

The first attempt used one `ScriptableRendererFeature` and tried to "skip" real rendering on warp frames by setting `cam.cullingMask = dummyLayer` (a layer with no objects). It worked in the Editor but produced **black frames in Play mode**.

Root cause: URP's RenderGraph compiler **prunes passes whose outputs aren't consumed**. When the scene has nothing to draw, nothing downstream depends on the capture/warp pass outputs, so the whole graph gets culled and the camera target is cleared to black. The Editor keeps extra overlays/gizmos alive that kept the chain live — masking the bug.

### The fix: two renderers, swap per frame

The solution is two `Universal Renderer` assets on the URP Asset:

| Index | Name | Features attached | Runs on |
| --- | --- | --- | --- |
| 0 | Normal Renderer | `ASWDualRendererFeature` (Mode = **Capture**) + usual stack | Real frames |
| 1 | Warp Renderer | `ASWDualRendererFeature` (Mode = **Warp**) only | Warp frames |

On each `LateUpdate`, the coordinator calls:

```csharp
var data = cam.GetUniversalAdditionalCameraData();
data.SetRenderer(isWarpFrame ? 1 : 0);
cam.cullingMask = isWarpFrame ? 0 : savedMask;
```

Why this avoids the black-frame issue: the Warp renderer's **only** pass writes directly to `activeColorTexture` with `builder.AllowPassCulling(false)`. The RenderGraph compiler cannot prune it — the camera backbuffer is a hard consumer. The standard opaque/transparent/shadow passes still *exist* in the Warp renderer but produce zero draw calls because culling mask is 0, so they cost essentially nothing.

### Why `LateUpdate` and not `beginCameraRendering`

Initial implementation called `SetRenderer()` inside `RenderPipelineManager.beginCameraRendering`. This threw:

```
InvalidOperationException: Type UnityEngine.Rendering.Universal.UniversalCameraData has already been created.
```

`SetRenderer()` triggers URP to rebuild the camera's per-frame `ContextContainer`. By the time `beginCameraRendering` fires, URP has already started populating it — the rebuild tries to `Create<UniversalCameraData>()` a second time on the same container, which throws.

`LateUpdate` runs **before** the render pipeline kicks in for the frame, so the swap happens while `ContextContainer` is empty. Safe.

---

## 4. `ASWDualRendererFeature.cs` in detail

### 4.1 Feature shell

```csharp
public class ASWDualRendererFeature : ScriptableRendererFeature
{
    public enum Mode { Capture, Warp }

    [Serializable]
    public class Settings
    {
        public Mode mode = Mode.Capture;
        public ComputeShader warpShader;
        public bool enableASW = true;
        public string cameraName = "Main Camera";
        public int warpRendererIndex = 1;
    }

    // Shared state between the two feature instances (one per renderer):
    internal static RTHandle s_HistColor;
    internal static RTHandle s_HistDepth;
    internal static RTHandle s_HistMotion;
    private static ASWDualRendererCoordinator s_Coord;
}
```

Key points:

- **One class, two instances.** The same feature is added to both renderers, disambiguated by `Mode`. This avoids the maintenance cost of two parallel feature classes.
- **Static history RTs.** Because the two feature instances live on different renderers, they can't share instance fields. `static RTHandle`s carry color/depth/motion across the capture frame (renderer 0) and the next warp frame (renderer 1).
- **Auto-spawned coordinator.** `Create()` on the Capture instance lazily instantiates a hidden `GameObject` holding `ASWDualRendererCoordinator`. The user doesn't manually place anything in the scene.

### 4.2 Pass wiring

```csharp
public override void AddRenderPasses(ScriptableRenderer renderer, ref RenderingData renderingData)
{
    if (!settings.enableASW) return;
    if (renderingData.cameraData.cameraType != CameraType.Game) return;

    if (settings.mode == Mode.Capture)
    {
        if (ASWDualRendererCoordinator.IsWarpFrame) return; // capture only on real frames
        _capturePass.ConfigureInput(ScriptableRenderPassInput.Color
                                  | ScriptableRenderPassInput.Depth
                                  | ScriptableRenderPassInput.Motion);
        renderer.EnqueuePass(_capturePass);
    }
    else
    {
        if (!ASWDualRendererCoordinator.IsWarpFrame) return; // warp only on warp frames
        renderer.EnqueuePass(_warpPass);
    }
}
```

`ConfigureInput(..Motion)` is important: it tells URP to generate motion vectors for this camera. Without it, `resources.motionVectorColor` is invalid.

### 4.3 Capture pass — saving the real frame

Runs at `RenderPassEvent.AfterRenderingPostProcessing` so it captures the **final** post-processed color. It MRTs color/depth/motion into the three static `RTHandle`s using `Hidden/ASW/HistoryCapture`:

```csharp
builder.SetRenderAttachment(dstColor,  0, AccessFlags.Write);
builder.SetRenderAttachment(dstDepth,  1, AccessFlags.Write);
builder.SetRenderAttachment(dstMotion, 2, AccessFlags.Write);

builder.SetRenderFunc(static (PassData d, RasterGraphContext ctx) =>
{
    ctx.cmd.SetGlobalTexture("_ASW_SrcColor",  d.srcColor);
    ctx.cmd.SetGlobalTexture("_ASW_SrcDepth",  d.srcDepth);
    if (d.srcMotion.IsValid())
        ctx.cmd.SetGlobalTexture("_ASW_SrcMotion", d.srcMotion);
    Blitter.BlitTexture(ctx.cmd, d.srcColor, new Vector4(1, 1, 0, 0), d.mat, 0);
});
```

Formats are chosen to match what the warp kernel expects:

| History RT | Format | Why |
| --- | --- | --- |
| `s_HistColor` | `R8G8B8A8_UNorm` | Matches back-buffer; random-write for the compute pass |
| `s_HistDepth` | `R32_SFloat` | Linearized depth; full float for accurate occlusion |
| `s_HistMotion` | `R16G16_SFloat` | URP native motion-vector format (screen-space UV delta) |

### 4.4 Warp pass — synthesising the extrapolated frame

Runs at `RenderPassEvent.BeforeRenderingOpaques` on the Warp renderer. There are no opaques to draw, so it effectively owns the whole frame.

```csharp
public override void RecordRenderGraph(RenderGraph rg, ContextContainer frameData)
{
    if (s_HistColor == null) return; // nothing captured yet

    var res = frameData.Get<UniversalResourceData>();
    var cam = frameData.Get<UniversalCameraData>();
    int w = cam.cameraTargetDescriptor.width;
    int h = cam.cameraTargetDescriptor.height;

    // Two transient RG textures:
    TextureHandle hAtomic = rg.CreateTexture(new TextureDesc(w, h) {
        colorFormat = GraphicsFormat.R32_UInt,
        enableRandomWrite = true,
        name = "ASW_AtomicBuffer" });

    TextureHandle hOutput = rg.CreateTexture(new TextureDesc(w, h) {
        colorFormat = GraphicsFormat.R8G8B8A8_UNorm,
        enableRandomWrite = true,
        name = "ASW_WarpedOutput" });
```

Then two sub-passes:

1. **Compute pass** dispatches `ClearBuffer`, `ScatterFrame`, `ResolveAndFillHoles` sequentially on `hAtomic` / `hOutput`.
2. **Raster blit pass** copies `hOutput` to `res.activeColorTexture` so it becomes the presented frame.

The blit is what keeps the whole chain alive — it's the consumer that prevents RenderGraph from pruning everything upstream.

### 4.5 Coordinator

```csharp
public class ASWDualRendererCoordinator : MonoBehaviour
{
    public static bool IsWarpFrame { get; private set; }

    void LateUpdate()
    {
        var cam = FindCamera();
        if (cam == null) return;

        bool on = _enabled == null || _enabled();
        IsWarpFrame = on && (Time.frameCount % 2 != 0);

        var data = cam.GetUniversalAdditionalCameraData();
        if (IsWarpFrame) {
            data.SetRenderer(_warpRendererIndex);
            if (!_hasSavedMask) { _savedCullingMask = cam.cullingMask; _hasSavedMask = true; }
            cam.cullingMask = 0;
        } else {
            data.SetRenderer(0);
            if (_hasSavedMask) { cam.cullingMask = _savedCullingMask; _hasSavedMask = false; }
        }
    }
}
```

Responsibilities:

- Decide `IsWarpFrame` based on frame parity.
- Swap the camera's active URP renderer.
- Zero out `cullingMask` on warp frames so CullingResults has no renderers to iterate → cheap broadphase.
- Save/restore the original culling mask when leaving warp mode.

It's the **only** piece of per-frame glue logic in the entire feature. Everything else is declarative.

---

## 5. `ASWWarp.compute` in detail

Three kernels, all `[numthreads(8,8,1)]`. Full-screen dispatch: `ceil(w/8) × ceil(h/8) × 1`.

### 5.1 Kernel 1 — `ClearBuffer`

```hlsl
RWTexture2D<uint> _AtomicBuffer;

[numthreads(8, 8, 1)]
void ClearBuffer(uint3 id : SV_DispatchThreadID)
{
    if (id.x >= (uint)_ScreenParams.x || id.y >= (uint)_ScreenParams.y) return;
    _AtomicBuffer[id.xy] = 0;
}
```

Zeros the atomic buffer. `0` is reserved as "no scatter landed here".

### 5.2 Kernel 2 — `ScatterFrame` (the core of ASW)

```hlsl
float2 motion = _PrevMotion[id.xy];

// Correction A: motion covers 2 real-frame intervals because we render every other frame.
// For the intermediate warp frame, advance only half the motion.
motion *= 0.5;

// Correction B: URP motion Y sign differs from our UV convention on D3D11.
motion.y = -motion.y;

float2 oldUV = (float2(id.xy) + 0.5) * _ScreenParams.zw;
float2 newUV = oldUV + motion;
if (newUV.x < 0 || newUV.x > 1 || newUV.y < 0 || newUV.y > 1) return;

uint2 newID = uint2(newUV * _ScreenParams.xy);

// Pack source pixel coords into a single uint:
//   bit 31        : valid (always 1)
//   bits 30..16   : source X (14 bits → up to 16383)
//   bits 15..0    : source Y (14 bits)
uint validBit  = 1u << 31;
uint packedData = validBit | ((id.x & 0x3FFF) << 16) | (id.y & 0x3FFF);

InterlockedMax(_AtomicBuffer[newID], packedData);
```

This is **forward scatter**: each thread represents a pixel in the *previous* frame and writes *forward* to where that pixel should appear in the *new* frame. Multiple source pixels can land in the same destination — hence the atomic.

Known caveats we identified:

- `InterlockedMax` is currently used only as a generic "last-writer-wins with a stable tiebreak". Depth is computed (`_PrevDepth[id.xy]`) but **not** actually packed into the high bits — so disocclusion ordering is arbitrary. For a real implementation, depth should occupy bits 30..22 (with nearer depth having a larger value so `InterlockedMax` picks the foreground), and coords move to bits 21..0.
- `motion *= 0.5` and `motion.y = -motion.y` are **hard-coded** — should be uniforms passed from C# to keep the kernel portable across graphics APIs and frame-rate ratios.
- `_ScreenParams` shadows Unity's built-in global of the same name — rename to `_ASW_ScreenParams` before shipping.

### 5.3 Kernel 3 — `ResolveAndFillHoles`

```hlsl
uint packedData = _AtomicBuffer[id.xy];

if (packedData != 0)
{
    uint2 srcID = uint2((packedData >> 16) & 0x3FFF, packedData & 0x3FFF);
    _OutputColor[id.xy] = _PrevColor[srcID];
}
else
{
    // 3x3 neighborhood average of any valid neighbours
    float4 colorSum = 0;
    int count = 0;
    for (int dy = -1; dy <= 1; dy++)
    for (int dx = -1; dx <= 1; dx++) {
        int2 nb = int2(id.xy) + int2(dx, dy);
        if (nb.x < 0 || nb.y < 0 || nb.x >= (int)w || nb.y >= (int)h) continue;
        uint nbPacked = _AtomicBuffer[nb];
        if (nbPacked != 0) {
            uint2 nSrcID = uint2((nbPacked >> 16) & 0x3FFF, nbPacked & 0x3FFF);
            colorSum += _PrevColor[nSrcID];
            count++;
        }
    }
    _OutputColor[id.xy] = count > 0 ? colorSum / (float)count : float4(0, 0, 0, 1);
}
```

Two responsibilities in one kernel:

1. **Resolve** — if this pixel received a scatter hit, look up the source color in `_PrevColor` using the packed source coordinate.
2. **Hole fill** — if it didn't, disocclusion happened (a foreground object moved and revealed background that wasn't in the previous frame). Average any valid 3×3 neighbours as a first-approximation inpaint.

This is the biggest optimisation target: the 3×3 loop runs for every pixel even when only a tiny fraction are holes. Splitting into two kernels (resolve-only, then hole-fill with an early-out on already-valid pixels) is the standard fix.

---

## 6. `ASWHistoryCapture.shader` in detail

A tiny full-screen blit shader whose single job is to copy three source textures (current color, depth, motion) into the three persistent history `RTHandle`s in **one** MRT draw call. Without this shader, the capture pass would need three separate blits.

### 6.1 Why a custom shader is needed

`Blitter.BlitTexture` on its own can only write to a single render target. The capture pass sets up three MRT attachments:

```csharp
builder.SetRenderAttachment(dstColor,  0, AccessFlags.Write);
builder.SetRenderAttachment(dstDepth,  1, AccessFlags.Write);
builder.SetRenderAttachment(dstMotion, 2, AccessFlags.Write);
```

To populate all three with one draw, the fragment shader must emit `SV_Target0/1/2` — which a generic blit shader can't do. Hence `Hidden/ASW/HistoryCapture`.

### 6.2 Shader source

```hlsl
Shader "Hidden/ASW/HistoryCapture"
{
    SubShader
    {
        Tags { "RenderType"="Opaque" "RenderPipeline"="UniversalPipeline" }

        Pass
        {
            Name "HistoryCaptureMRT"
            ZTest Always Cull Off ZWrite Off

            HLSLPROGRAM
            #pragma vertex Vert
            #pragma fragment Frag

            #include "Packages/com.unity.render-pipelines.universal/ShaderLibrary/Core.hlsl"
            #include "Packages/com.unity.render-pipelines.core/Runtime/Utilities/Blit.hlsl"

            TEXTURE2D(_ASW_SrcColor);
            TEXTURE2D(_ASW_SrcDepth);
            TEXTURE2D(_ASW_SrcMotion);

            struct FragmentOutput
            {
                float4 color  : SV_Target0;
                float4 depth  : SV_Target1;
                float4 motion : SV_Target2;
            };

            FragmentOutput Frag(Varyings input)
            {
                FragmentOutput o;
                float2 uv = input.texcoord;

                o.color  = SAMPLE_TEXTURE2D(_ASW_SrcColor,  sampler_PointClamp, uv);
                o.depth  = SAMPLE_TEXTURE2D(_ASW_SrcDepth,  sampler_PointClamp, uv).r;
                float2 mv = SAMPLE_TEXTURE2D(_ASW_SrcMotion, sampler_PointClamp, uv).rg;
                o.motion = float4(mv, 0, 0);
                return o;
            }
            ENDHLSL
        }
    }
}
```

### 6.3 Noteworthy details

- **Reuses URP's Blitter `Vert`** via `#include ".../Blit.hlsl"`. This gives a free fullscreen triangle vertex stage with correct `Varyings.texcoord` handling across platforms and — crucially — **stereo eye-index plumbing** for XR. Writing a vertex shader manually would risk breaking single-pass instanced stereo.
- **Reads globals, not `_BlitTexture`.** Blitter's own convention is one input via `_BlitTexture`. Because we need three inputs, C# binds them as globals (`_ASW_SrcColor/Depth/Motion`) and the shader samples those. The `Blitter.BlitTexture` call on the C# side is still used — but only to drive the geometry and state setup; its source texture argument is effectively ignored by this fragment.
- **`sampler_PointClamp`.** Point sampling is mandatory here: bilinear would corrupt depth values (they aren't linear across edges) and motion vectors (averaging two opposing MVs produces zero on edges — a worst-case artefact for the scatter pass).
- **`ZTest Always Cull Off ZWrite Off`.** Standard fullscreen-blit state. Depth is written as a *color* value to `SV_Target1` (an R32_SFloat target), not to a depth attachment — so `ZWrite Off` is correct.
- **Depth is written to an `R32_SFloat` color attachment**, not a depth buffer. The fragment output type is `float4` only because HLSL MRT structs prefer uniform types; only `.r` carries meaning and that's the only channel the compute shader reads (`Texture2D<float>`).
- **Motion `.rg` → `float4(mv, 0, 0)`.** URP's motion-vector texture format is `R16G16_SFloat` (two channels, UV delta). Written back into the `R16G16_SFloat` history RT via the `.rg` of this `float4`; `.ba` are discarded.

### 6.4 Capture data flow

```
URP current frame          ASWHistoryCapture.shader              Static history RTs
──────────────────         ──────────────────────────            ──────────────────────────
activeColorTexture  ──▶  _ASW_SrcColor  ──▶ SV_Target0  ──▶  s_HistColor  (R8G8B8A8_UNorm)
cameraDepthTexture  ──▶  _ASW_SrcDepth  ──▶ SV_Target1  ──▶  s_HistDepth  (R32_SFloat)
motionVectorColor   ──▶  _ASW_SrcMotion ──▶ SV_Target2  ──▶  s_HistMotion (R16G16_SFloat)
```

One draw, three outputs. The compute shader on the next (warp) frame samples those three RTs as `_PrevColor / _PrevDepth / _PrevMotion`.

---

## 7. Why the Motion correction `* 0.5` matters

This is the single most confusing aspect of this POC and worth a slide on its own.

URP's motion vectors encode **"how far did this pixel move from the previous rendered frame to this one"**.

- In a normal game at 60 Hz, that's one 1/60 s interval.
- In our POC, because we skip odd frames, the *rendered* frames are 1/30 s apart. URP's MVs therefore describe motion across **two** display frames.
- When we extrapolate the *intermediate* warp frame, we want to predict motion for **one** display-frame interval — half of what URP gave us.

Hence `motion *= 0.5`. For a real runtime integration, this scale factor must equal `(display Hz / app Hz) − 1` — e.g. at 90 Hz display / 45 Hz app, scale = 1.0 (warp frame is exactly halfway). On a mock project hard-coded `0.5` is fine.

---

## 8. Data flow summary (per frame pair)

```
┌─ Real frame (frame N, even) ───────────────────────────────────┐
│                                                                │
│  URP Renderer 0                                                │
│    ├─ Opaque / Transparent / PostFX ...                        │
│    └─ ASW Capture Pass (AfterRenderingPostProcessing)          │
│         → writes s_HistColor, s_HistDepth, s_HistMotion        │
│    → Present                                                   │
└────────────────────────────────────────────────────────────────┘

┌─ Warp frame (frame N+1, odd) ──────────────────────────────────┐
│                                                                │
│  LateUpdate:                                                   │
│    SetRenderer(1)                                              │
│    cullingMask = 0                                             │
│                                                                │
│  URP Renderer 1 (warp-only)                                    │
│    ├─ (Opaque / Transparent: zero draws, culled)               │
│    └─ ASW Warp Pass (BeforeRenderingOpaques)                   │
│         ├─ Clear _AtomicBuffer                                 │
│         ├─ Scatter: _PrevMotion + _PrevDepth → _AtomicBuffer   │
│         ├─ Resolve: _AtomicBuffer + _PrevColor → _Output       │
│         └─ Blit _Output → activeColorTexture                   │
│    → Present                                                   │
└────────────────────────────────────────────────────────────────┘
```

---

## 9. Performance observations (profiler)

Intermittent 30–50 ms spikes observed during playtime. Suspect areas and investigation path:

| Symptom | Likely cause | Tool to confirm |
| --- | --- | --- |
| Main thread spike only | `RTHandles.Alloc` firing on resize, or `SetRenderer` triggering URP rebuild every frame | Deep Profile; log inside `EnsureHistoryRTs` |
| GPU spike only | `InterlockedMax` atomic contention (large MVs cluster many sources to one destination) | RenderDoc per-dispatch timing |
| Both CPU+GPU spike | RenderGraph recompile due to pass-topology flip each frame | Profiler search `RenderGraph.Compile` |
| Periodic 33.3 ms / 50 ms | V-Sync quantization, not our code | Disable V-Sync and retest |
| First-frame spike only | Shader variant compile | Normal, warm up at startup |

Two cheap wins to try first:
1. Only call `SetRenderer()` when the index actually **changes** (currently called every `LateUpdate`).
2. Keep *both* passes enqueued every frame and branch inside `SetRenderFunc` on a uniform — prevents RenderGraph topology flipping and the recompile cost.

---

## 10. Path to a real XR runtime integration

The mock is structured so the *algorithm* carries over verbatim; only the surrounding plumbing changes.

### What stays the same

- **Scatter → Resolve → Fill** kernel structure.
- **R32_UInt atomic buffer** as the rendezvous between source and destination.
- **Packed source-coordinate** representation (with depth added in the high bits for a real build).
- **Motion scale** as a uniform `= displayHz / appHz − 1`.

### What changes

| Mock (this project) | Real runtime |
| --- | --- |
| `ASWDualRendererCoordinator.LateUpdate` swaps renderer every other frame | Runtime compositor decides when to warp based on **missed frame detection** (app submission latency > budget) |
| App renders every frame; we fake skipping via culling mask 0 | App explicitly renders at half display rate; runtime receives a submit timestamp and schedules warps in gaps |
| `Capture Pass` writes to `RTHandle` static fields | App submits color + depth + motion textures to the runtime via the XR swapchain (`XR_FB_space_warp` extension on Meta) |
| Warp pass runs as a URP `ComputePass` inside the app's frame | Warp runs in the **runtime / compositor process**, outside the app, on its own queue, usually a reprojection + warp combined pass |
| `s_HistColor/Depth/Motion` are plain `RTHandle`s | Shared textures exposed through the XR swapchain — the app writes, the compositor reads; requires cross-process sync (timeline semaphores / fences) |
| Output blit to `activeColorTexture` | Output composited by the runtime with runtime UI, overlays, distortion correction, chromatic aberration, etc. |
| Single mono camera | Stereo — run scatter/resolve per eye; motion includes head reprojection, not just per-object motion |

### Concrete migration steps (suggested order)

1. **Factor the compute kernels** out of the URP render feature into a standalone compute library callable from any frame-graph system. They already are — `ASWWarp.compute` has no URP-specific code.
2. **Promote packing to include depth** so `InterlockedMax` gives correct occlusion.
3. **Expose motion scale, Y-flip, and resolution** as constant-buffer uniforms. Remove all hard-coded values.
4. **Add reprojection from camera motion**, not just rasterized per-object motion — for HMD head pose, sample a per-frame `ΔPose` matrix and compose it with the object MV. This is what ATW+ASW hybrids do.
5. **Split `ResolveAndFillHoles`** into two dispatches and reduce hole-fill cost with a groupshared tile.
6. **Replace `RTHandle` history** with a swapchain-backed shared resource once inside the runtime. Add cross-queue synchronisation.
7. **Replace frame-parity logic** with the runtime's frame-scheduling signal (e.g. `xrWaitFrame` / compositor's missed-frame flag).
8. **Stereo**: dispatch kernels per eye, with a per-eye atomic buffer.

### Open problems that the real runtime must solve and the mock doesn't

- **Disocclusion quality** — a 3×3 average is visible as blur; real runtimes use iterative push-pull or neural inpaint.
- **Foreground/background separation** — requires correct depth packing + possibly per-layer warping.
- **Alpha-blended geometry** — motion vectors are undefined for transparent objects; runtime may need to mark them and skip warping.
- **UI / HUD layers** — should be rendered on the warp frame (at full rate) *after* warping, never warped. The mock doesn't handle this.
- **Timing** — on a real HMD, the warp frame has a hard budget (≈ 2–4 ms on mobile XR). Our scatter + 3×3 fill is far outside that.

---

## 11. TL;DR slide

- **Problem:** fake Oculus ASW in Unity URP without any XR runtime.
- **Core idea:** skip rendering every other frame; synthesise that frame by forward-warping the previous color with motion + depth.
- **Unity-specific trick:** use **two URP renderers** swapped per frame via `UniversalAdditionalCameraData.SetRenderer()` — a single-renderer + empty-culling-mask approach is silently pruned by RenderGraph and produces black frames.
- **Compute pipeline:** `ClearBuffer` → `ScatterFrame` (atomic packed scatter) → `ResolveAndFillHoles` (lookup + 3×3 inpaint).
- **Known sharp edges:** motion scale must match `displayHz/appHz`; `InterlockedMax` needs depth packing for correct occlusion; resolve + fill should be split; `_ScreenParams` name collision with Unity's built-in.
- **Migration:** algorithm is runtime-ready; plumbing (swapchain, stereo, scheduling, sync) is the delta.
