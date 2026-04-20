---
title: "Mob-FGSR: Frame Generation and Super Resolution for Mobile Real-Time Rendering"
author:
  - "Sipeng Yang"
  - "Qingchuan Zhu"
  - "Junhao Zhuge"
  - "Qiang Qiu"
  - "Chen Li"
  - "Yuzhong Yan"
  - "Huihui Xu"
  - "Ling-Qi Yan"
  - "Xiaogang Jin"
source: https://mob-fgsr.github.io/
created: 2026-04-20
tags:
  - mobile-rendering
  - frame-generation
  - super-resolution
  - motion-vectors
  - real-time-rendering
  - siggraph-2024
  - no-neural-net
  - splatting
status: true
ingested: 2026-04-20
---
SIGGRAPH 2024

[Paper](https://mob-fgsr.github.io/resources/paper/SIGGRAPH_Conf_Mob_FGSR.pdf) [Supp.](https://mob-fgsr.github.io/resources/paper/Sup_Mob_FGSR_Super_Resolution_and_Frame_Generation_for__Mobile_Real_time_Rendering.pdf) [Code](https://github.com/Mob-FGSR/MobFGSR) [Commercial Application](#application)

![](https://mob-fgsr.github.io/resources/images/teaser.png)

Requiring only color, depth, and motion vectors (MVs) from frames rendered at times 0 (A) and 1 (C), our approach efficiently produces high-quality interpolated (B) / extrapolated (D) frames and their SR counterparts at desired times.

## Abstract

Recent advances in supersampling for frame generation and super-resolution improve real-time rendering performance significantly. However, because these methods rely heavily on the most recent features of high-end GPUs, they are impractical for mobile platforms, which are limited by lower GPU capabilities and a lack of optical flow estimation hardware. We propose MobFGSR, a novel lightweight supersampling framework tailored for mobiledevices that integrates frame generation with super resolution to effectively improve real-time rendering performance. Our method introduces a splatbased motion vectors reconstruction method, which allows for accurate pixel-level motion estimation for both interpolation and extrapolation at desired times without the need for high-end GPUs or rendering data from generated frames. Subsequently, fast image generation models are designed to construct interpolated or extrapolated frames and improve resolution, providing users with a plethora of options. Our runtime models operate without the use of neural networks, ensuring their applicability to mobile devices. Extensive testing shows that our framework outperforms other lightweight solutions and rivals the performance of algorithms designed specifically for high-end GPUs. Our model’s minimal runtime is confirmed by on-device testing, demonstrating its potential to benefit a wide range of mobile real-time rendering applications.

## Method Overview

![](https://mob-fgsr.github.io/resources/images/pipeline.png)

For the **interpolation model**, MVs and depth from I-frames 0 and 1 are used for motion splatting to generate bidirectional MVs 𝑀 <sub>0→𝛼</sub> and 𝑀 <sub>1→𝛼</sub>. The color and depth data from these frames are then warped and blended to create the interpolated frame.  
For the **extrapolation model**, we predict single-direction MVs 𝑀 <sub>1→1+𝛼</sub> and apply a disocclusion filling module to handle missing MVs in disoccluded areas. The extrapolated frame is produced by performing backward warping on frame 1.

## Motion Estimation

![](https://mob-fgsr.github.io/resources/images/motion_estimation.png)

By analyzing two consecutive frames, we estimate precise pixel motions along linear and nonlinear trajectories with uniform acceleration (quadratic motion).

Depth-aware motion splatting with atomic operations ensures foreground pixels are prioritized in MVs.

![](https://mob-fgsr.github.io/resources/images/mv_splats.png) ![](https://mob-fgsr.github.io/resources/images/ColorImage.png)

Color image

![](https://mob-fgsr.github.io/resources/images/SplattedMVs.png)

Splatted MVs

![](https://mob-fgsr.github.io/resources/images/RefinedMVs.png)

Refined MVs

![](https://mob-fgsr.github.io/resources/images/ReferenceMVs.png)

Reference MVs

Splatted MVs often exhibit grid-like gaps, which are identified using a thin-object detection method and filled using a mean filter.

## Frame Reconstruction

For **interpolation**, we use reconstructed MVs to warp adjacent rendered frames and blend them for interpolation.

![](https://mob-fgsr.github.io/resources/images/Warped0.png)

Warped frame 0

![](https://mob-fgsr.github.io/resources/images/Warped1.png)

Warped frame 1

![](https://mob-fgsr.github.io/resources/images/Constructed.png)

Constructed frame

![](https://mob-fgsr.github.io/resources/images/Reference.png)

Reference

![](https://mob-fgsr.github.io/resources/images/frame_blend.png)

Use depth (D) and brightness (B) to determine whether changes are shading changes or disocclusions.

(1) Color is preferentially sampled from the temporally closer frame when the color and depth of the two warped frames are similar. (2) For disocclusions, appropriate pixels from both frames are selected to avoid ghosting artifacts. (3) For shading changes, a simple linear interpolation (lerp) of the two frames ensures a smooth transition.

For **extrapolation**, due to the lack of subsequent frame references, we warp a single frame and use a simplified disocclusion filling method to address gaps. This approach, utilizing MVs from the rendered frame, balances runtime efficiency and image quality effectively.

## Super Resolution

![](https://mob-fgsr.github.io/resources/images/super_resolution.png)

We introduced learned resampling weights for repeated image warping on mobile platforms. This data-driven approach learns optimal filters for repeated sampling, utilizing a lookup table (LUT) for rapid access to 16-pixel weights based on sampling positions (d <sub>x</sub> and d <sub>y</sub>). Experimental results indicate that the LUT-based method matches bicubic quality but operates faster.

![](https://mob-fgsr.github.io/resources/images/resample.png) ![](https://mob-fgsr.github.io/resources/images/resample_original.png)

Original

![](https://mob-fgsr.github.io/resources/images/resample_bilinear.png)

Bilinear

![](https://mob-fgsr.github.io/resources/images/resample_bicubic.png)

Bicubic

![](https://mob-fgsr.github.io/resources/images/resample_LUT.png)

LUT method

Our SR module employs a standard framework that reuses accumulated temporal samples from history frames to construct the high-resolution current image. To construct the SR frame, we align the history SR frame with the current frame using LUT-based image backward warping, rectify invalid pixels in the warped history frame, and blend it with the low-resolution frame to produce the final output. This module can seamlessly replace the standard image warping module in the frame generation process.

![](https://mob-fgsr.github.io/resources/images/sr_blend.png)

q is the reconstructed pixel, a and b are parameters acquired using data-driven optimization, d <sub>s</sub> is the distance from the sampling point to the pixel, and q <sup>w</sup> is the historical pixel.

## Results and Comparisons

![](https://mob-fgsr.github.io/resources/images/fg_comparison.png)

Comparison of our models, Ours-I (interpolation) and Ours-E (extrapolation), with existing frame generation methods including 3DWarp \[Mark et al. 1997\], BSR \[Yang et al. 2011\], and AFME \[Holmes and Wicks 2020\].

![](https://mob-fgsr.github.io/resources/images/sr_comparison.png)

Comparison of our models, Ours-SR, Ours-ISR (interpolation & SR), Ours-ESR (extrapolation & SR), with existing super resolution methods including TSR \[Epic 2022\], FSR 2 \[AMD 2022\], and DLSS 2 \[Liu 2020\].

![](https://mob-fgsr.github.io/resources/images/unity_scene_comparison.png)

Unity scenes with forward shading: Street View (SV), Meadows (ME), Hilly Area (HA), and Dragon Park (DP).

![](https://mob-fgsr.github.io/resources/images/ue_scene_comparison.png)

UE scenes with deferred shading: Bunker (BK) and Western Town (WT).

## On-device Testing

We analyzed performance on devices equipped with the Qualcomm Snapdragon 8 Gen 3 processors. Our method rapidly delivers high-quality frame generation and SR, ideal for enhancing mobile real-time rendering.

![](https://mob-fgsr.github.io/resources/images/runtime.png) ![](https://mob-fgsr.github.io/resources/images/runtime_comparison.png)

We developed an Android real-time rendering application and implemented resolution-related no-operation computational tasks to simulate intensive rendering environments. Without supersampling, the frame rate achieves about 22 FPS. Generating two frames can increase the frame rate to approximately 50 FPS, and using 2× super-resolution can boost it to over 110 FPS.

![](https://mob-fgsr.github.io/resources/images/android_runtime.png)

## Android Demo

////////////////////////////////////////////////////////////////////////////////////////  
//////////// Android demo is available [here](https://mob-fgsr.github.io/resources/demo/MobFGSR.apk). ////////////  
////////////////////////////////////////////////////////////////////////////////////////

![NoneSS](https://mob-fgsr.github.io/resources/images/NoneSS.jpg)

None SS

![Interpolation](https://mob-fgsr.github.io/resources/images/Interpolation.jpg)

Interpolation

![Extrapolation](https://mob-fgsr.github.io/resources/images/Extrapolation.jpg)

Extrapolation

![InterpolationSR](https://mob-fgsr.github.io/resources/images/InterpolationSR.jpg)

Interpolation & SR

![ExtrapolationSR](https://mob-fgsr.github.io/resources/images/ExtrapolationSR.jpg)

Extrapolation & SR

<video src="https://github.com/Mob-FGSR/Mob-FGSR.github.io/blob/main/resources/videos/android_demo.mp4?raw=true" controls=""></video>

## Application in Commercial Products of Our Method

[OnePlus Ace 3 Pro](https://www.oneplus.com/), the first android smartphone able to run *Genshin Impact* at a stable 120 frames per second, released in June 2024.

![](https://mob-fgsr.github.io/resources/images/oneplus_cn.png)

Original Chinese

![](https://mob-fgsr.github.io/resources/images/oneplus_en.png)

Translated

Related links:

- OnePlus Ace 3 Pro frame prediction for *Genshin Impact*:
	- 2024-06-27 [https://www.oneplus.com/cn/ace-3-pro#anchor-2](https://www.oneplus.com/cn/ace-3-pro#anchor-2)
		- 2024-06-27 [https://www.bilibili.com/video/BV1vy411z7sB/](https://www.bilibili.com/video/BV1vy411z7sB/) at 51:35~55:00
- OnePlus 13 frame prediction for *Genshin Impact* and *Honkai: Star Rail*:
	- 2024-10-31 [https://www.oneplus.com/cn/13#custom-anchor-3](https://www.oneplus.com/cn/13#custom-anchor-3)
		- 2024-10-31 [https://www.bilibili.com/video/BV1wVSnYbEeb/](https://www.bilibili.com/video/BV1wVSnYbEeb/) at 28:05~29:30

## Citation

```
@inproceedings{yang2024mob,
  title={Mob-FGSR: Frame Generation and Super Resolution for Mobile Real-Time Rendering},
  author={Yang, Sipeng and Zhu, Qingchuan and Zhuge, Junhao and Qiu, Qiang and Li, Chen and Yan, Yuzhong and Xu, Huihui and Yan, Ling-Qi and Jin, Xiaogang},
  booktitle={ACM SIGGRAPH 2024 Conference Papers},
  pages={1--11},
  year={2024}
}
```

閃記

複製 LaTeX 公式