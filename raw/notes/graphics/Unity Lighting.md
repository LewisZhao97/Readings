---
title: "Unity Lighting"
date: 2026-03-20
type: note
tags:
  - graphics
  - unity
  - lighting
  - render-path
  - forward-rendering
  - deferred-rendering
---
# Render Path
## Forward
- 逐物体计算，Unity中实现的forward对灯光做了数量限制减少遍历次数。
- 会有overdraw，远处或物体背后的物体仍然要计算光照，但会被前面的片元重复绘制。
- drawcall数量大，shader复杂度高，多个物体收多个相同灯光影响，每一个物体都要再次计算。
## Forward+
- 逐像素计算。
- 带光源剔除的Forward，在depth prepass中预先处理。
- 将屏幕分为若干个tile，用compute shader计算每个tile与哪些灯光体积有重叠，形成灯光列表，shader着色时查询列表计算光照。
- 灯光分配的颗粒度下降。由per-object -> per-pixel / per-tile

 **Forward与Forward+都是在着色阶段就进行了光照计算。** 
## Deferred
- 把几何与光照分离为多pass。
- 通过光栅化将物体表面信息输出到Gbuffer中作为纹理存储，深度测试正常工作。
- 物体按任意顺序提交，经深度测试后剔除后面的片元，也就是留下能看到的像素。
- 光照pass中，场景中的灯光体积与屏幕空间的像素有交叉才会被纳入计算，与Gbuffer中的数据做光照计算。理论上灯光数量不会显著影响性能。
![[Pasted image 20260320163325.png]]
# Unity built-in light
### 单方向光源
- Directional Light
- Point Light
- Spot Light
### 面光源
- Area Light (Bake Only)
### 光源模式
- Realtime
- Mixed
- Baked
![[Pasted image 20260313144501.png]]

## Bake Lighting Mode
- Baked Indirect
	- 这种模式仅烘焙场景的间接光照，而直接光照和阴影则是实时计算的。这意味着，场景中所有标记为static（静态）的物体将烘焙其间接光照信息，而动态物体则依赖于实时计算的光照。Baked Indirect适合用于静态场景，因为它不能处理动态物体的光照变化。
- Subtractive
	- 静态物体的直接光和阴影都烘焙进 Lightmap，运行时几乎没有实时光照开销，是三者中最省性能的。但问题来了：动态角色怎么接收静态场景的阴影？Unity 用了一个"减法混合"的 hack——把场景烘焙颜色和实时光颜色做差值运算来近似阴影。这个近似在有色灯光或复杂光照下会产生明显的色差，角色走进阴影区颜色会"跳"一下。适合移动端这类对性能极度敏感、动态角色不需要和静态阴影精确交互的场景。
- Shadow Mask
	- 这是三者中质量最高的方案。它单独烘焙出一张 Shadow Mask 纹理（RGBA 四通道，可以存 4 盏灯的遮挡信息），静态物体之间的阴影从这张图里采样，完全精确；动态物体进入静态阴影区时，也是采样这张图来决定遮挡程度，不存在 Subtractive 的色差问题。运行时实时阴影只在近距离（Shadow Distance 范围内）生效，超出范围自动切换用 Shadow Mask 接管——形成无缝过渡。额外代价是这张 Shadow Mask 纹理本身的内存，以及每盏灯只能占用一个通道（最多同时影响 4 盏灯的遮挡），超出数量的灯会 fallback 回 Baked 模式。

## Light Probes
### 球谐函数（Spherical Harmonics）
- Light Probe 在烘焙时，会在探针所在位置将周围全景光照及颜色（间接光）信息烘焙成低频信息，保留颜色及亮暗。
- L2 阶球谐，27 个浮点数（9个系数 × RGB三通道）。
![[Pasted image 20260313171258.png]]
### Light Probe group
本质是在空间中手动放一堆采样点，烘焙时每个点记录一份球谐函数（SH，Spherical Harmonics）数据，用 9 个系数近似表示来自四面八方的光照。运行时动态物体会找到自身周围构成四面体的四个探针，对它们的 SH 做重心插值，得到当前位置的近似光照。

最大的痛点是**全靠手工**——探针稀疏的地方角色走过去光照会突然跳变，薄墙两侧的探针没有遮挡判断会互相污染，大型场景要摆几百上千个点工作量很大。
### Adaptive Probe Volume
APV 不需要手动摆放，系统根据场景几何体自动生成一个三维探针网格，靠近墙壁和物体的区域自动加密，空旷区域自动稀疏（这就是"Adaptive"的含义）。采样时不是四面体插值，而是在三维纹理空间里做连续插值，物体在任意位置移动都能平滑过渡。

它还内置了 **Validity 权重**机制——烘焙时每个探针会检测自己是否在几何体内部或墙后，如果是无效位置就降低采样权重，从相邻有效探针里借光，从根源上解决了漏光问题

[Counter Strike 2 光照探针场](https://www.bilibili.com/video/BV1C841117Lr/?vd_source=6f8de31cd39eef3e8e6411c629606bf4)