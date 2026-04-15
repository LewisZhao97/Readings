---
title: Space Warp with Depth Propagation in XR Applications
author:
  - Yingen Xiong
  - Christopher Peri
source: https://ieeexplore.ieee.org/document/9666145
created: 2026-04-14
tags:
  - xr
  - space-warp
  - depth-map
  - depth-propagation
  - hmd
  - remote-rendering
  - computer-graphics
status: true
ingested: 2026-04-14
---
Yingen Xiong

Samsung Research America

Mountain View, USA

e.xiong@samsung.com

Christopher Peri

Samsung Research America

Mountain View, USA

chris.peri@samsung.com

Abstract-In this paper, an image guided depth point extraction and depth map reconstruction approach is presented for generating high quality depth maps used for Space-Warp functions on XR devices. The approach can be divided into two parts. The first part is for adaptive depth point extraction to obtain sparse depth points which can efficiently represent the original depth map which runs on device side and the second part is for depth map recovery with the extracted depth points and Space-Warp with the recovered depth map which runs on HMD side. In adaptive depth point extraction, feature points, edges, object contours are extracted as a candidate point set which represents fast depth changes. A criterion is defined by depth map, color image, and spatial information to compute confidence of the candidate points. They are integrated together with the confidence map to build a depth point set. In depth map recovery, an image guided depth map reconstruction from the sparse depth points with depth propagation is presented. During depth map reconstruction, it keeps the sparse points unchanged and propagates the depths in the neighborhood points to the considered points guided by the color image and original depth map information. The reconstructed depth map is used for Space-Warp to create a new frame based on the current frame and a predicted head pose for the new frame. The algorithm is applied to XR systems and applications. The results demonstrate the advantages of the proposed approach. We compare results created by our algorithm and generally used approaches including bilinear interpolation and multilevel B-Spline interpolation. The comparison shows that our algorithm can create high-quality new frames with clear object boundaries and less artifacts.

Keywords-Adaptive Depth Point Extraction, Depth Map, Space-Warp, XR, AR, VR, Dense Depth Map Generation, Depth Map Densification, Depth Map from Scattered Data, Head Mounted Device (HMD)

# I. INTRODUCTION

With the introduction of 5G and other high speed wireless protocols, a new generation of XR devices are employing more and more compute off the HMD and onto remote device and/or cloud systems. This is especially with AR devices that need to be a small and lightweight as possible, however, a price in latency is extracted when moving information from the HMD to the device and back. To mitigate these issues, a technique known as late stage re-projection a.k.a Space-Warp [3], [8], [9] is employed to re-render an image to reflect changes in a users head pose in 6 DOF. Key to using Space-Warp is depth information. This creates a challenge. It is already expensive enough in time and power

![](7bbac493db6193e9482e6ba2ab3e141546c7d97c59e6a8826b8c7bde9f98a1cd.jpg)  
Figure 1. Depth Map Down-Scale and Up-Scale with a Regular Grid.

to move a rendered image from device to HMD, but to also send a depth map of the same resolution becomes double given the required accuracy of the depth map dataset. As such, it is critical to reduce the amount of depth information being sent as possible to meet time and power requirements while being able to reconstruct a full depth map with enough accuracy to perform the Space-Warp operation at a sufficient level of quality.

The process of down-scaling and up-scaling a depth map with regular grid approaches produces artifacts in the recovered depth map. Fig. 1 shows one of the examples. The left shows an original depth map, the middle shows a down-sampled depth map with scale factor 0.25 in width and height, and the right shows the result by up-scaling the down-sampled depth map to the original size. From the result, we can see that image features are smoothed, especially, edges and contours of objects appear zigzag shape, pixels are missing on these edges and contours, which causes artifacts when it is applied to Space-Warp.

Approaches for down-scaling and up-scaling a depth map with a regular grid does not consider image and object features. As such, taking simple bilinear averaging can miss important details needed for a good depth map.

In order to obtain a good recovered depth map, we need to determine which points can efficiently represent the original depth map. The corresponding color image to the depth map can provide information for extracting these high value points. In this paper, we present a superior approach from the typical sub-sampled depth point solution by determining high value depth points determined by employing a color image guider. We collect these high value points by extracting image feature points, edges, and contours of the objects.

There are a lot of approaches for image feature detection and extraction [2], [4], [5], [7], [10]. In SIFT [7] and SURF [2], Keypoints of objects are detected and extracted in multi-scale levels for object detection and recognition. These feature points are supposed to efficiently represent the objects in the images. Object edges in the images can be efficiently detected and extracted by Canny edge detection approach [4]. These object edges can provide information about how the object shape and depth changes. Lee et al [10] give a good survey of edge detection approaches. Like object edges, object contours [5] are also very important in representing object shapes and depth changes. In this paper, we detect and extract image keypoints, object edges, and object contours and integrate them together as a set of depth points on device side. We transmit the point set to HMD for reconstructing the full depth map.

On the HMD side, after receiving a depth point set from the device side, we need to reconstruct the full depth map for Space-Warp in XR applications. Since the depth point set contains discrete point data, the reconstruction of full depth map from the point set belongs to surface reconstruction from spare data. There are various approaches to interpolating sparse data such as linear triangular interpolation, cubic triangular interpolation, triangle based blending interpolation, inverse distance weighted methods, radial basis function methods, and natural neighbor interpolation methods. Lee et al use multilevel B-Splines to interpolate the sparse data to obtain dense depths [6]. Amidror [1] gives an excellent survey of these categories of approaches. The interpolation algorithms can be used for obtaining dense depth maps conveniently, however, this kind of approaches may create artifacts in the resulting depth maps including blurring object boundaries, false depths, and missing depths on object contours. In this paper, we propose an image guided dense depth map recovery approach to reconstruct the dense depth map from extracted depth points. In this approach, we propagate depths in the neighborhood points with guiders of color image and original depth map information. We can reconstruct the dense depth map with clear object boundaries and contours and reduce noises in the extracted point set. We describe the approach in detail in next sections.

We select sparse depth points from the depth map guided by image feature detection and extraction on the device side and reconstruct a depth map from the sparse depth points with the image guided depth propagation approach. The work in this paper includes,

- Detect and extract image feature points, object edges, and object contours;   
- Verify the extracted these feature points, edges, and contours with the depth map;   
- Integrate these extracted points with the depths to create a sparse point set to make them efficiently represented the original depth map;   
- Reconstruct a depth map with the sparse point set with

![](2e50564c75663211df343d2538af9b004e21e7617e1963590116f143a895455b.jpg)  
Figure 2. Workflow of the adaptive depth point extraction.

the image guided depth propagation approach;

- Apply the reconstructed depth map to Space-Warp for XR applications;   
- Demonstrate advantages of the proposed algorithm with applications in XR systems.

Here is the structure of this paper. In Section II, we introduce the workflow of our approach. The details of adaptive depth point extraction, dense depth map reconstruction, and Space-Warp transformation are described in Section III. Applications and result analysis are discussed in Section IV. A summary of the paper is given in Section V.

# II. WORK FLOW OF THE APPROACH

Fig. 2 shows a workflow of extracting sparse depth points. Firstly, feature points, object edges, and object contours are extracted with feature detection approaches. Secondly, these feature points, edges, and object contours are integrated with an adaptive algorithm to create a set of sparse feature points. Combining with the corresponding depth map, we can obtain a set of sparse depth points. Meanwhile, we label the contours of objects, so that these labels can be used in some useful applications such as object recognition, occlusion, 3D analysis. Finally, the sparse depth points are transmitted from the device side to the HMD side through WiFi.

Fig. 3 shows a workflow of Space-Warp with depth propagation. In this process, a dense depth map is recovered with depth propagation and applied to Space-Warp to create a new frame. Firstly, the HMD receives a set of sparse depth points from the device and gets the current RGB frame image. Secondly, a data term is created with the depth at the considered pixel and the depth of the sparse points. With the data term, we intent to keep the sparse points unchanged. A smooth term is created with the depth

![](a513a1a75faf5db7d50e789284711dfb868030c087074db805faea15527b75eb.jpg)  
Figure 3. Workflow of Space-Warp with Depth Propagation.

at the considered pixel and the depths at neighborhood pixels. With the smooth term, we propagate the depths at the neighborhood points to the considered pixel. In the smooth term, the weight is computed from color image and original depth map information. We optimize the criterion function to obtain depths for all pixels. Finally, we can recover a dense depth map using the receive sparse depth points and the current RGB frame image.

After reconstruction of the dense depth map, we perform Space-Warp image re-projection to create a new frame. Firstly, we normalize the RGB-D coordinate and create an homogeneous coordinate. We transform the 2D frame to 3D space with the head pose and the predicted pose of the next frame to obtain a new frame. This image re-projection process is commonly known as Space-Warp.

The detailed algorithms of sparse depth point extraction, dense depth map recovery, and re-projection of Space-Warp, will be described in following sections.

# III. SPACE-WARP WITH ADAPTIVE DEPTH POINT EXTRACTION AND DENSE DEPTH RECONSTRUCTION

# A. Problem Expression

Applications of traditional depth interpolation solutions which compute a higher-resolution depth map by employing a weighted average of neighboring points from a lower-resolution depth map can result in blurred edges and blurred object contours, which causes artifacts when performing reprojection of Space-Warp on XR devices.

Our goal is to achieve a higher-resolution depth map with significantly improved preservation of object edge and contour information. Since there is typically a high correlation between color image correspondence with depth map information, we can determine object edge and contour information, allowing us to compute the higher-resolution depth map. In the meantime, we can reduce noise during depth reconstruction.

With the reconstructed high-resolution and high-quality dense depth map, we can create a high-quality re-projected

![](63f8ffc764af3b385f4a27a244f70e2ce5576e8f37948c085ac0c9eefda15ed7.jpg)

![](ba0415d6a87b415dee7dc63abfe7764318cde147228fb933e7261b8ff9027c10.jpg)

![](a656452a619624e2e152b2971949945a9350feaa8a65342b43266a808815bd7f.jpg)  
Figure 4. Example of Original Color Image and Depth Map.

![](c47c936b7be6e86028390236c4536efba8fbc812b45d0a0f5042bc6892e181c2.jpg)  
Figure 5. Feature Point and Edge Extraction.

frame by Space-Warp operation.

# B. Adaptive Depth Point Extraction

Fig. 4 shows an example color image on the left and a corresponding depth map on the right created with a 3D modeling application 'Blender'. We use them as source data to describe the algorithm and applications in the paper.

In multi-scale feature point extraction and verification, we extract points at which the pixel intensities and depths suddenly change and create a fast approach for the extraction process. We design a multi-scale feature point extraction algorithm to extract these points.

In multi-scale feature point extraction, we extract feature points in each scale level. We use Gaussian scale space to create Gaussian image pyramids. For each level, we extract feature points with Difference of Gaussian(DOG) or Laplacian of Gaussian (LOG) filter.

We integrate the extracted feature points of all scale levels together and verify each point with depth information. We remove the points whose depths have little or no change compared with neighboring points.

With the multi-scale extraction, integration, and verification process, we can obtain good image feature points which best represent depth changes which can be used for efficiently recovering the dense depth map. The left of Fig. 5 shows results of multi-scale feature point extraction and verification.

There is another kind of image features which can efficiently represent the image. It is image edges. We detect and extract edge features in the image and we care about

![](be9746ed465bdfc0da95339ddebbffc0f670fea129119b3a458cc3781f76c17b.jpg)

![](1b698b7398fa35448d8c901fd89faa238e12737850dcc173da808c681c96ce36.jpg)  
Figure 6. Image and Object Contour Detection and Extraction.

more to object edges in the image. Further more, we extract object edge points whose depths have changes.

We create multi-scale images such as image pyramids and detect and extract edge features at each scale level. For example, the edge feature extract can use Canny edge filter, Sobel edge filter, or other filters.

We integrate the edge feature points of all scale levels together and verify the edge feature points with their depths. We remove the edge points whose depths have little or no change compared with their neighboring pixels.

The right of Fig. 5 shows an example results of multiscale edge detection and verification. These edge features efficiently represent object shape changes with depth changes.

In the process of object and image contour detection and verification, we detect and extract object and image contours and their depths in the image so that we can recover the depth map for the objects efficiently.

We detect and extract object and image contours and remove noise points by removing the ones whose depths do not change or change a little. The left of Fig. 6 shows the results of object contour detection and extraction and the right of Fig. 6 shows the results of image contour detection and extraction. The image contour can also be used to generate mask for dense depth map recovery.

In the process of sparse depth point integration and verification, we integrate all feature points obtained with previous described approaches to create a sparse depth point set for dense depth map recovery.

We integrate the feature points obtained from multi-scale feature detection and extraction, multi-scale edge detection and extraction, and object and image contour detection and extraction together to create a sparse point set.

During the integration process, we check point relative positions and remove duplicated points and the ones which are too close each other. We verify the depths for the integrated points again to remove the ones whose depths have little or no change compared with their neighborhoods.

We divide the integrated depth points into multiple levels with confidence scores which are calculated from previous detection and extraction processes. According to WiFi transmission requirements, we can combine different levels to provide sparse depth point sets so that we can control the

![](27933bcb272224b733c799698a0a04b1bc5f60e48fbbf927dbbd6183b28d597a.jpg)  
Figure 7. Sparse Depth Point Integration and Verification.

number of points in the point set. For example, when WiFi condition is good, we can transmit more points. When WiFi condition is not good, we can reduce the number of points in transmission. Further, for complex vs. simple scenes, we can vary the sparsity level accordingly. Fig. 7 shows an example result for sparse depth point integration and verification.

# C. Image Guided Dense Depth Map Recovery

Since the original depth map is created by a synthetic rendering system, we can assume that depths in the original depth map are accurate. Since the sparse depths are selected from the original depth maps, they did not change unless they may lose some accuracy during encoding/decoding and transmission from the device side to the HMD side. Our goal for depth map recovery on HMD is to keep the sparse depths unchanged and propagate depths in neighboring pixels to the considered points guided by color image and original depth map information.

We create a criterion for depth propagation with a data term and a smooth term,

$$
\begin{array}{l} J (d) = \mu_ {s} W _ {s} \sum_ {p} | | d (p) - d _ {s} (p) | | _ {2} ^ {2} \\ \text {w h e r e}, \quad + \mu_ {m} \sum_ {(p, q) \in N} W _ {p q} | | d (p) - d (q) | | _ {2} ^ {2}, \tag {1} \\ \end{array}
$$

$\mu_{s}$ is a coefficient for balancing the data term with the smooth team;   
$W_{s}$ is the weight for using sparse points;   
$d(p)$ is the depth at the current considered pixel $p$ ;   
$d_{s}(p)$ is the depth at a sparse depth point;   
$\mu_{m}$ is a coefficient for balancing the smooth term with the data team;   
$W_{pq}$ is the weight for using color image and original depth map information;   
$d(q)$ is the depth at pixel q;   
$N$ is neighborhood.

We minimize the criterion function $J(d)$ to obtain depths,

$$
d (p) = \underset {d} {\arg \min } J (d), \tag {2}
$$

where, $d(p)$ is the depth at pixel $p$ .

# D. Depth Based Space-Warp

Space-Warp is used for re-projecting the current 2D frame to 3D space with depths and a predicted head pose to create a new frame, such that we can save computational power and maintain target frame-rates. The procedure can be described below.

We convert the screen coordinate to a normalized device coordinate,

$$
\left( \begin{array}{c} x _ {n d c} \\ y _ {n d c} \\ z _ {n d c} \end{array} \right) = \left( \begin{array}{c} 2 \frac {x _ {s c}}{w} - 1 \\ 2 \frac {y _ {s c}}{h} - 1 \\ d \end{array} \right), \tag {3}
$$

where,

$(x_{ndc},y_{ndc},z_{ndc})^T$ is a normalized device coordinate; $(x_{sc},y_{sc})^T$ is a screen coordinate.

$w$ is width of the image frame;

$h$ is the height of the image frame;

$d$ is the depth.

We compute the re-projected coordinate with

$$
\left( \begin{array}{c} x _ {p} \\ y _ {p} \\ z _ {p} \\ w _ {p} \end{array} \right) = P S _ {i} S _ {r} ^ {- 1} P ^ {- 1} \left( \begin{array}{c} x _ {n d c} \\ y _ {n d c} \\ z _ {n d c} \\ 1 \end{array} \right), \tag {4}
$$

where,

$(x_{p},y_{p},z_{p})^{T}$ is the re-projected coordinate;

$w_{p}$ is a scale factor;

$P$ is projection matrix;

$S_{i}$ is the predicted head pose for the new frame;

$S_{r}$ is the head pose of the current frame.

We re-project all pixels in the original frame to create a new frame. Since the re-projection is based on depths and new head pose, holes will be created when color information is not available from the original frame. We created an algorithm to fill these holes. The results shown in next sections are obtained by Space-Warp and hole filling operations.

# IV. APPLICATIONS AND RESULT ANALYSIS

The proposed Space-Warp with depth propagation algorithm has been applied to XR systems and implemented with OpenGL ES. The approach has been tested with different XR scenes. Good results have been obtained. In this section, we present some exemplar applications and results which are obtained on Samsung Note 10 plus phone with Exynos 9825 (7 nm) - EMEA/LATAM and RAM 12 GB. We also tested on other Samsung phones including S10, Note10, S20, S20+.

![](2267f26cc5f209b718360a11f1b3139b0595eff89f05e8957d131716d3b14482.jpg)

![](69e3e6fb3cea0bd3458f25e253728ed9f3ffe9d077ddf913b96dc0ad746e2697.jpg)

![](ee48fc6d46f097d0728837a44a1c3564e2a580842fa24727142235c0ac487f9d.jpg)

![](06c18085ec84ab89a07eae9c57e03c22bae7c70072418060e12f1cce98f568aa.jpg)  
Figure 8. Down-sampling depth images.

# A. Creation of Original Color Images and Depth Maps

In order to simulate the XR pipeline, we created color images and depth maps of different scenes as ground truth data for the tests. Fig. 4 shows an example of the test color image and depth map. In the figure, the left shows the full-resolution color image $(1024\times 1024)$ and the right shows the full-resolution depth map $(1024\times 1024)$ .

In this dataset, the color image is in RGBA format and the depth map is a grayscale image. For the depth map, the camera near and far planes are 0.037 and 0.083 meters respectively. In the depth image, pixel brightness is relative to the distance camera where 1 is far, 0 is near.

# B. Depth Map Downsampling

In order to test the image guided depth propagation algorithm, we down-sample the ground truth depth map with different scale factors. Fig. 8 shows the grids with different levels of down-sampling factors. The order of top left, top right, bottom left, and bottom right shows the down-sampling points for scale factors 0.125 (16384 points), 0.0625 (4096 points), 0.03125 (1820 points), and 0.015625 (1024 points) respectively.

# C. Sparse Depth Points

We extract image features including feature points, image edges, object contours, image contours and integrated them together to create sparse depth points from the source depth map. We use same amounts of depth points as the downsampling with standard grids for the source data of multilevel B-spline interpolation and depth propagation. Fig. 9 shows sparse depth points extracted from source depth map. The order of top left, top right, bottom left, and bottom right

![](ef41caf98ed012d607bc968d5887edd08be1f74536dad2ecb59bd245113613cf.jpg)

![](fef6732d9186384bd8c7bb4593950eb7c8c110268c790770fe7751376526bec6.jpg)

![](803abbe1e4f48e4626d808dabac2afc02e240c6b05c85a0ae89cc6b13c1b9d26.jpg)

![](09c8119978baf071318f022b0177d015b23b5ab54fb4b22c2441463af307a82c.jpg)  
Figure 9. Sparse Depth Point Extraction from the Source Depth Map.

shows the sparse point numbers 16512, 4435, 1858, 1062 respectively, which corresponds to the point numbers in Fig. 8 with the same order. We will use the sparse depths to reconstruct depth maps and generate Space-Warp images.

# D. Depth Map Reconstruction

In order to make comparison of results between our depth propagation and traditional interpolation, we generate a higher-resolution depth map with bilinear interpolation. Fig. 10 shows the results. The order of top left, top right, bottom left, and bottom right shows interpolation results for scale factors 0.125, 0.0625, 0.03125, and 0.015625 respectively. From the results we can see that interpolated higher-resolution depth maps are blurred and some points in object edges and contours are missing. As the scale factor becomes smaller, the interpolation result is getting more blurred and more missing pixels on edges and contours of the objects, which will cause more artifacts in 3D object reconstruction and 3D re-projection.

We also reconstruct the depth map with multilevel B-Spline interpolation for more comparison with the results generated by the proposed algorithm in this paper. Fig. 11 shows the results. The order of top left, top right, bottom left, and bottom right shows interpolation results for numbers of sparse depth points 16512, 4435, 1858, 1062 respectively.

We reconstruct the dense depth map with the proposed depth propagation algorithm. Fig. 12 shows the results. The order of top left, top right, bottom left, and bottom right shows interpolation results for numbers of sparse depth points 16512, 4435, 1858, 1062 respectively.

By comparing the results of bilinear interpolation shown in Fig. 10 and depth propagation shown in Fig. 12, we

![](cb2612e872b5f6045630230b90225634658218fbbbce973c3e72be38ae627e19.jpg)

![](a7e75f7478ed28a72542da2cd3830092ba2175a926b30e90e90e93afe23274ea.jpg)

![](283529c25bee319a5cb0229a72a53d077c095fc8629343d4e9d9e565d259e174.jpg)

![](fd81086b89a2ab89b338a9f5510c0a6faaf06c5cb79bae2d141827ad199aa589.jpg)  
Figure 10. Interpolation with bilinear approach.

![](771e2c8898edcf879e7bf21b3831bbb16002dfbdcecea501abbe9c945f4b9673.jpg)

![](b348148dbc149b79f96d10aaaea7564417db93026848a7335d26551e3ab041d7.jpg)

![](d44a0a71889a03136b056483a3668358e105129883a13f259171100a0f7ac5ea.jpg)

![](62c2fddfedf8d0e2b2a5548238b7dc450c13ecaf6c5d3ab4614e37c4067652f0.jpg)  
Figure 11. Interpolation with B-Spline approach.

can see that the results generated by bilinear interpolation have too many artifacts in which points on boundaries are missing and objects are blurred inside the depth map. Even in the best case which is reconstructed from downscale factor 0.125, there still exist boundary and blurring issues. On the other hand, the results generated by depth propagation are much better than that of bilinear interpolation. The reconstructed depth maps have very clear boundaries and very clean objects. In the cases of reconstruction from 16512

![](e3994f6b2d46c51e645be8aad36d85478e5fb03e808ce3739a6ed321322ab7ae.jpg)  
Figure 12. Depth reconstruction with Depth Propagation.

points and 4435 points, the reconstructed depth maps are very similar as the original depth map. As the number of sample points is reduced, some artifacts appear inside the depth maps. However, they are still much better than that of bilinear interpolation.

By comparing the results of multilevel B-spline interpolation shown in Fig. 11 and depth propagation shown in Fig. 12, we can see that the results generated by multilevel B-spline interpolation have too many artifacts in which the depths inside the objects are not well reconstructed. The depth distributions inside the objects are very different from the original depth map. As the number of sparse points decreases, the artifacts inside the depth maps increase and the quality of the depth map becomes worse and worse. On the other hand, the results generated by depth propagation are much better, which is described above. The quality of depth maps generated by depth propagation is much better than that of multilevel B-spline interpolation.

# E. Space-Warp with the Dense Depth Map

We apply the dense depth maps reconstructed by depth propagation, bilinear interpolation, and multilevel B-Spline interpolation to Space-Warp to create new frames and compare the results to demonstrate the advantages of our proposed algorithm.

Fig. 13 shows the results of Space-Warp with the dense depth maps created by bilinear interpolation shown in Fig. 10. The order is the same as Fig. 10. From the results we can see that similar as the reconstructed depth maps, pixels on the boundaries are missing and straight lines are not straight anymore after re-projection and there exist many other artifacts inside the images.

![](f217822332c474228f65e0fb9eb1cecbb275bbae2024991b128bc5cb328de35a.jpg)  
Figure 13. Space-warp with depths generated with bilinear interpolation.

Fig. 14 shows the results of Space-Warp with the depth maps created by multilevel B-Spline interpolation shown in Fig. 11. The order is the same as Fig. 11. From the results we can see that similar as the reconstructed depth maps, straight lines are not straight after re-projection and there exist many other artifacts inside the images. There are also artifacts on the boundaries of objects.

Fig. 15 shows the results of Space-Warp with the depth maps created by depth propagation shown in Fig. 12. Like the above cases, the order is the same as Fig. 12. Here we can see that the results are much better than that of bilinear interpolation and multilevel B-Spline interpolation. They have very clear boundaries and object shapes. The straight lines of the boundaries keep straight after re-projection. The results of the cases (16494 points and 4190 points) are very similar as the original color image.

In conclusion, the depth maps and color images generated by depth propagation are better than that generated with multilevel B-spline interpolation and bilinear interpolation. The depth propagation reconstruction can generate high quality Space-Warp results.

# V. DISCUSSION AND CONCLUSIONS

We have presented a depth propagation algorithm and solution which takes an important role to maintain target frame rates in XR applications while reducing power and time requirements for data transmission. Different from traditional approaches such as bilinear and multilevel B-Spline interpolation which only use spatial information from neighboring points, the image guided depth propagation approach uses spatial, intensity, and depth information to construct

![](242ec864ab924041ae0259e35a55c58e445cd3370616ef488c6d27550d66b827.jpg)

![](17c781fbde6419136babdb317c31dab29fdd1fdde8618bd3fb4f3a4a8816177a.jpg)

![](fab3b99a52af9269e5966689f458e21d0ae22423285346161e7db318cf2898ed.jpg)

![](65f263b89761332f38e44599947fcd331692e7a68a0b42fcdc7e22d04083ab74.jpg)  
Figure 14. Space-warp with depths generated with B-Spline interpolation.

![](e8515a52e8c8d2ebc999ef927eb78e96c7bea0aa700e28f7214b73f327c1bc4a.jpg)

![](038532b071c11413e2329e63522afaba305bb5841f4be8ee9e2366191704e047.jpg)

![](55a250a858d045c03f1f5e70ecb773e087eab320a8247f3675042da9e81c68c7.jpg)

![](433aad0de0d30650f0f2c98af96d462a66c0bcfa3629a75f478f2a1fa83ce347.jpg)  
Figure 15. Space-warp with depth map generated with depth propagation.

depths of the considered points. Depths at considered points are propagated from depths of neighborhood points. The reconstructed depth map is applied to a Space-Warp function to generate new frames from updated poses based on reference frames. In order to make a comparison, we also generate new frames with depth maps created with bilinear interpolation and multilevel B-Spline interpolation. From the results, we can see that depth maps generated by the image guided depth propagation algorithm and color images

generated by Space-Warp using the depth maps are well reconstructed. Comparing with the results generated with bilinear interpolation and multilevel B-Spline interpolation, the depth maps and color images are clearer and edges and contours of objects are well preserved. The color images have better visual quality.

The main disadvantage of the algorithm comparing with bilinear approach is that the computational cost is higher, however, in cases where lag between the rendered images and display time on the HMD is significant, the added computational cost is less than the added cost of sending higher resolution depth information.

# REFERENCES

[1] I. Amidror, "Scattered data interpolation methods for electronic imaging systems: a survey," Journal of Electronic Imaging, vol. 11, no. 2, pp. 157-176, 2002.   
[2] H. Bay, A. Ess, and T. T. L. V. Gool, "Surf: Speeded up robust features," Computer Vision and Image Understanding (CVIU), vol. 110, no. 3, pp. 346-359, 2008.   
[3] D. Beeler, E. Hutchins, and P. Pedriana. (2016) Asynchronous spacewarp. [Online]. Available: https://developer.oculus.com/blog/asynchronous-spacewarp/   
[4] J. Canny, “A computational approach to edge detection,” IEEE Transactions on Pattern Analysis and Machine Intelligence, vol. 8, no. 6, pp. 679–698, 1986.   
[5] X. Gong, H. Su, Z. Z. D. Xu, F. Shen, and H. Yang, "An overview of contour detection approaches," International Journal of Automation and Computing, vol. 15, no. 6, pp. 656-672, 2018.   
[6] S. Lee, G. Wolberg, and S. Y. Shin, "Scattered data interpolation with multilevel b-splines," IEEE Transactions on Visualization and Computer Graphics, vol. 3, no. 3, pp. 228-244, 1997.   
[7] D. Lowe. (1999, Sep.) Object recognition from local scale-invariant features. Kerkyra, Corfu, Greece.   
[8] L. McMillan and G. Bishop, "Plenoptic modeling: An image-based rendering system," in SIGGRAPH '95: Proceedings of the 22nd annual conference on Computer graphics and interactive techniques, Los Angeles California, USA, Sep. 1995, pp. 39-46.   
[9] W. M. snd L. McMillan and G. Bishop, "Post-rendering 3d warping," in Proceedings of the 1997 symposium on Interactive 3D graphics, Providence Rhode Island, USA, Apr. 1997, pp. 7-16.   
[10] D. Ziou and S. Tabbone, "Edge detection techniques: An overview," International Journal of Pattern Recognition and Image Analysis, vol. 8, no. 4, pp. 537-559, 1998.