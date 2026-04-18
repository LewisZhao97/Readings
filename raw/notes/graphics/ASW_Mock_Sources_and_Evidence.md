# ASW Mock — Sources & Evidence

Follow-up to [`ASW_Mock_TechSharing.md`](ASW_Mock_TechSharing.md). Captures the two questions raised during review of the mock:

1. Where does the **two-renderer swap** pattern come from — any paper or authoritative source?
2. Where does **forward reprojection (scatter + resolve)** come from — proof that ASW should be built this way?

Short version: the two-renderer swap is Unity plumbing, not theory. Forward scatter is theory, well-sourced. Keep them mentally separate when presenting this work.

---

## 1. The two-renderer swap has no academic source

It's an engineering assembly of three documented Unity primitives. There is no paper, SDK reference, or Meta documentation that prescribes it. It was arrived at by debugging the RenderGraph pruning failure.

The individual pieces that the pattern rests on are each documented:

| Piece | Source |
| --- | --- |
| A URP asset can hold a list of Universal Renderers, and a camera can pick any of them at runtime | Unity Scripting API: `UniversalAdditionalCameraData.SetRenderer(int index)` — *"Takes an index that maps to the list on the Render Pipeline Asset … dynamically assigns a different ScriptableRenderer to the camera."* |
| RenderGraph automatically culls passes whose outputs aren't consumed | Unity Graphics repo, `com.unity.render-pipelines.core/Documentation~/render-graph-write-render-pass.md`: *"The RenderGraph system offers efficient resource management and automatically culls unused render passes."* |
| `AllowPassCulling(false)` is an escape hatch, not a production solution | same doc: *"It is important to note that this should not be used in production code, as it can prevent the render graph system from optimizing by removing unneeded render passes."* |
| Writing to an imported backbuffer keeps the writer alive | same doc's canonical example: `ImportBackbuffer(BuiltinRenderTextureType.CameraTarget, ...)` + `SetRenderAttachment(cameraTarget, 0, AccessFlags.Write)` |
| Multiple-renderer-per-camera is a known community pattern for special render modes | Daniel Ilett's URP portal renderer and similar tutorials use separate Universal Renderers per camera role |

### Why this matters when presenting the work

Don't overstate it. The two-renderer shape is one of several valid ways to keep the capture/warp passes un-culled. Equally valid:

- One renderer with an **explicit backbuffer consumer** that is always bound (the warp pass on warp frames, a pass-through blit on real frames).
- One renderer with `AllowPassCulling(false)` on the warp pass (pragma non grata per Unity's own doc, but works).

The mock picks two-renderer because it reads cleanest: each renderer has one job, no branching inside a single feature, and the standard URP graph stays unmodified on real frames.

### Verification, not proof

You can't "prove" this via paper citation, but you can verify the behavior on your machine:

- **Window → Analysis → Rendering Debugger → Render Graph Viewer** (URP ≥ 14). Run the broken single-renderer variant: the capture/warp nodes are marked *culled*. Run the two-renderer variant: the warp node writes to the imported CameraColor and stays live.
- Frame Debugger alone is insufficient — it only shows the compiled graph after culling.

### Separately, on the `InvalidOperationException`

The `UniversalCameraData has already been created` throw when calling `SetRenderer()` in `beginCameraRendering` is a consequence of `ContextContainer`'s lifecycle: URP creates the per-frame container once per camera-render. `SetRenderer()` triggers a rebuild which calls `Create<T>()` a second time on the same container, which throws. `LateUpdate` precedes the render loop for the frame, so the swap lands on an empty container. This is derivable from URP's rendering-loop ordering; not a bug, just an ordering constraint.

---

## 2. Forward reprojection (scatter + resolve) is well-sourced

### Meta's own documentation (primary source)

From *Application SpaceWarp — How it Works* (`developers.meta.com/horizon/documentation/unity/os-app-spacewarp/`):

> *"predict where the pixel will be in the next frame … by moving pixels to their predicted location in the synthesis frame"*

> *"motion vectors [are] the NDC space position difference between the current frame and previous frame for corresponding pixels … how much the pixel moved in screen space"*

That is forward scatter by definition: the motion vector is **anchored at the source pixel** and dispatches that pixel to its destination. Combined with Positional TimeWarp for the 6-DOF head-motion component.

From *Application SpaceWarp for Unity* (`developers.meta.com/horizon/documentation/unity/unity-asw/`):

> *"We are writing the 3D motion vector data to the R, G, and B channels in the color texture."*

> *"the difference of the previous frame's vertex position transformed down to clip space"*

App-side MotionVecPass matches the mock's structure.

### OpenXR extension (standardized surface)

`XR_FB_space_warp` is the Khronos-standardized extension for app→runtime ASW. App submits **color + motion-vector texture + depth** per real frame; runtime performs the warp. The mock's input shape (color/depth/motion) and pipeline split (app produces, "runtime" extrapolates) deliberately mirror this.

### Academic lineage

- **Mark, W. R., McMillan, L., & Bishop, G. (1997).** *Post-Rendering 3D Warping.* I3D '97. Canonical depth-image forward-warping paper. Every modern Space-Warp descends from this.
- **McMillan, L., & Bishop, G. (1995).** *Plenoptic Modeling: An Image-Based Rendering System.* SIGGRAPH '95. Introduces the **occlusion-compatible scan order** that pre-GPU forward warpers used to resolve multi-source-to-one-destination ordering. Modern implementations (including the mock) replace this with a depth-packed `InterlockedMax` on a uint atomic buffer.
- **Xiong & Peri 2021**, **Peri & Xiong 2021** (already in the KB) — both build on the same forward-reprojection formulation and address the split-rendering depth-bandwidth problem.

### Why scatter, not gather — three independent reasons

1. **Direction of the known function.** The motion field is `f: source → destination`. Gather needs `f⁻¹`, which is ill-posed under disocclusion — multi-valued where occluders overlap, undefined on revealed background. Scatter uses the function in its defined direction; the resolve/hole-fill step handles the many-to-one and zero-to-one cases explicitly.
2. **Depth ordering is a write conflict.** Foreground should win over background when multiple sources land in the same destination. `InterlockedMax` on a depth-packed uint is the GPU-era implementation of McMillan's scan order. Gather can't express this natively — it'd require a prior pass to build `f⁻¹` with z-buffering, which *is* a scatter pass in disguise.
3. **Extrapolation vs interpolation.** DLSS 3 Frame Generation and FSR 3 do gather because they interpolate between two known frames with bidirectional optical flow available. ASW extrapolates forward from one frame — only unidirectional (source→destination) motion is available, so gather's prerequisite is absent.

### Caveats on the mock's specifics

The *algorithm shape* (forward scatter + atomic resolve + hole fill) is academically sound and industry-confirmed. The *specific kernel details* are pragmatic placeholders flagged in the original note's section 10:

- `InterlockedMax` currently uses valid-bit + coords only. For correct occlusion, the atomic's high bits should encode depth (nearer depth = larger value, so max picks foreground), coords in the low bits.
- 3×3-average hole fill is a first-pass placeholder. Real runtimes use push-pull pyramids or learned inpainting; joint bilateral (see [[Graphics - Bilateral Filter]]) is a cheap upgrade step.
- `motion *= 0.5` is hard-coded. The correct formulation is `scale = (displayHz / appHz) − 1`, per the note.
- `_ScreenParams` shadows Unity's built-in global — rename before shipping.

---

## 3. What to do with this material

- `wiki/entities/XR - Space-Warp.md` has been updated with:
  - The 1997/1995 academic citations (Mark/McMillan/Bishop).
  - A *Scatter, not Gather* section summarizing the three-reasons argument.
  - Industry citations: Meta AppSW docs and `XR_FB_space_warp`.
- No entity or topic page was created for the two-renderer pattern itself — it's a Unity-plumbing idiom, not a reusable concept worth a KB page. If the project needs its own tech note, it lives next to the mock code, not in the wiki.

## 4. Sidebar from today — Karpathy Autoresearch

Separately ingested [Karpathy's autoresearch repo](https://github.com/karpathy/autoresearch); summary at `wiki/summaries/LLM - Autoresearch.md`. Conclusion after reviewing the repo with GitHub MCP: **don't run it, read `program.md`.** It's a single-GPU LLM-training sandbox with no direct application to this graphics work. The interesting artifact is `program.md` as a reference for writing autonomous-agent "skill" files — branch-per-run, TSV log, advance-or-reset semantics, simplicity criterion, NEVER-STOP clause — transplantable to any closed-loop XR sub-problem that has a headless oracle (shader tuning, upscaler parameter search, bake quality trade-offs). Not transplantable to anything requiring human judgement in-headset.

---

## References

- Unity Scripting API, `UniversalAdditionalCameraData.SetRenderer`, `docs.unity3d.com/Packages/com.unity.render-pipelines.universal@17.0/api/UnityEngine.Rendering.Universal.UniversalAdditionalCameraData.html`.
- Unity Graphics, `Packages/com.unity.render-pipelines.core/Documentation~/render-graph-write-render-pass.md` (RenderGraph auto-culling, `AllowPassCulling`).
- Meta, *Application SpaceWarp — How it Works*, `developers.meta.com/horizon/documentation/unity/os-app-spacewarp/`.
- Meta, *Application SpaceWarp for Unity*, `developers.meta.com/horizon/documentation/unity/unity-asw/`.
- Khronos, `XR_FB_space_warp` OpenXR extension specification.
- Mark, W. R., McMillan, L., & Bishop, G. (1997). *Post-Rendering 3D Warping*. I3D '97.
- McMillan, L., & Bishop, G. (1995). *Plenoptic Modeling: An Image-Based Rendering System*. SIGGRAPH '95.
