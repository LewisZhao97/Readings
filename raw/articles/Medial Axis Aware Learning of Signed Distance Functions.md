---
title: "Medial Axis Aware Learning of Signed Distance Functions"
author:
  - "Samuel Weidemaier"
  - "Christoph Norden-Smoch"
  - "Martin Rumpf"
source: "https://arxiv.org/html/2604.16512v1"
created: 2026-04-22
tags:
  - neural-sdf
  - signed-distance-function
  - medial-axis
  - phase-field
  - ambrosio-tortorelli
  - eikonal-equation
  - geometry-processing
  - surface-reconstruction
status: true
ingested: 2026-04-22
---
Samuel Weidemaier Institute for Numerical Simulation, University of Bonn Christoph Norden-Smoch Institute for Numerical Simulation, University of Bonn Martin Rumpf Institute for Numerical Simulation, University of Bonn

## 1 Introduction

Computing the signed distance function (SDF) to an unoriented point cloud by learning a neural network is a challenging task that has been addressed frequently in recent years, cf. [^30] [^22] [^5] [^35] [^33] [^31] [^8] [^32] [^34]. It involves designing loss functionals such that the underlying surface can be represented accurately by the zero-level set, and at the same time, estimating the (signed) distance to the surface well.

Applications of SDFs range from collision detection [^23] and constructive solid geometry [^25] to the use of *level set methods* for solving partial differential equations (PDEs) on surfaces [^26] [^34]. Furthermore, SDFs enable efficient ray tracing and rendering of implicit surfaces [^16].

It is well known that the SDF is the *viscosity* solution to the eikonal equation, a first-order hyperbolic PDE, which makes it difficult to design appropriate loss functionals. The structure of the eikonal equation implies that solutions grow linearly in direction normal to the surface. Already for smooth closed surfaces, this leads to the fact that there exists a set, where the gradient of the SDF has to jump (cf. fig.˜2). This set is the *medial axis* of the surface, i.e., the lower-dimensional set, where the nearest-point projection onto the surface is not well-defined. Hence, a global regularization with a higher-order loss functional, including the medial axis, is not possible.

In addition to an eikonal loss, we introduce a second-order loss functional that encourages the approximate neural SDF to grow linearly in normal direction, together with a neural phase field function that detects the medial axis, where the gradient of the SDF jumps, and switches off the regularizer in those regions (cf. fig.˜2). This phase field is controlled by a loss functional of Ambrosio-Tortorelli-type, known in the context of phase field approximations of the Mumford-Shah functional for image segmentation. Our method computes a global neural network approximation of the signed distance function that is accurate both in the near field, allowing for high fidelity surface reconstruction, and in the far field. Experimental results demonstrate that our approach achieves competitive or superior performance compared to state-of-the-art methods across both local and global metrics.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/box_example.jpg)

Figure 2: Left: level sets of the neural SDF ϕ \\phi of a hexaeder. Middle: zero-level set of the SDF. Right: phase field approximation of the jump set of the SDF gradient, which coincides with the hexaeder’s medial axis.

### Related work

##### Neural implicit representations.

Implicit representations are a flexible tool in geometry processing [^11]. Especially the implicit representation by the signed distance function is useful, in particular for the applications described above. The method SIREN [^30] uses neural networks with sine activation function, together with an eikonal loss, to compute the SDF of a surface given by a point cloud. In [^22], the PHASE method is introduced to compute a neural implicit representation of a surface using an occupancy function, by minimizing the Modica-Mortola energy [^27]. The resulting phase field function, represented by a neural network, exhibits two distinct phases corresponding to the interior and exterior regions of the surface. The phase transition occurs in a narrow band around the surface defined by the input point cloud. In a post-processing step, the phase field function can be used to approximate the SDF, using a $\log$ -transform.

##### Neural higher order variational SDF methods.

The method DiGS [^5] modified the SIREN [^30] approach to also include a second-order derivative loss functional, namely the squared $L^{2}$ -norm of the Laplace operator. The singular structure of the Hessian of the SDF, induced by the linear growth in normal direction, was used in [^33] and [^31], by introducing quadratic loss functionals depending on the determinant, and the second derivative in normal direction, respectively, in the near field of the surface. StEik [^35] also made use of this fact and introduced a second-order $L^{1}$ -loss for the directional divergence in normal direction.

##### Heat methods for computing SDFs.

Another approach for computing approximate distances, introduced by Crane et al. [^9] [^10], leverages the heat equation for geometry processing. This is based on a two-step method, by first solving the heat equation for a small time step, with the target object as the heat source. The gradients of the corresponding heat solution are then used to compute an approximate distance in a second step. In [^14], this approach was generalized to compute SDFs, starting from an oriented point cloud. HeatSDF [^34] adapts this two-step method in the learning context to compute an approximate neural SDF from *unoriented* point clouds, incorporating additional constraints in order to orient the gradients. The HotSpot method [^32] also uses the relation between the heat equation and distances in a one-step algorithm, by introducing an additional loss together with an eikonal loss, such that the SDF is encouraged to be the logarithm of the solution to the heat equation.

##### Higher order phase transition theory.

In 1987, Aviles and Giga [^3] proposed a functional of Modica-Mortola-type [^27], with higher order derivatives, in the context of smectic liquid crystal theory and conjectured the sharp-interface limit to be a functional that depends on the jump set of the gradient of a function that solves the eikonal equation almost everywhere. In two dimensions, the appropriate function space for the limit energy was investigated in [^1], and the asymptotic sharp lower bound was studied in [^18]. In [^21], an example was given that the theory is fundamentally different in higher space dimensions. In [^4], a connection of the minimizer of a sharp-interface limit energy to the viscosity solution of the eikonal equation was drawn.

##### Ambrosio-Tortorelli phase field models.

The Mumford-Shah functional, cf. [^28], is well-known in the context of image segmentation, allowing jumps of the image segmentation function along interfaces, and controlling the co-dimension one measure of the jump set. The phase field functional introduced by Ambrosio and Tortorelli, cf. [^2], allows to approximate minimizers of the Mumford-Shah functional by smooth functions, where the jump set is approximated by a phase field function. In [^15], a similar phase field model was introduced to approximate brittle fracture in the context of elasticity. Numerical approximation by finite elements was first discussed in [^7]. Recently, a discretization of a phase field fracture model based on a deep-Ritz method for neural networks was proposed in [^24].

### Our contribution

- We introduce a variational approach to compute the global signed distance function of a surface represented by an unoriented point cloud and the discontinuity set of its gradient, i.e., the medial axis of the surface (cf. fig.˜2). The cost functional measures the second-order derivative in normal direction where the SDF is smooth. Furthermore, it controls the $(d-1)$ -dimensional measure of the discontinuity set.
- We use an Ambrosio-Tortorelli type phase field model to variationally describe the medial axis together with a proper weighting of the second-order term.
- The method is implemented using neural networks for both the SDF and the phase field, which are trained simultaneously, allowing for an efficient computation.
- We perform a quantitative evaluation and a comparison with other approaches for 2D and 3D databases, showing the high approximation quality of the SDF.

## 2 Motivation

At first, we discuss different properties of the signed distance function of a closed, compact surface which will motivate the different components of our variational method. We assume that the surface $\mathcal{S}=\partial\omega$ is the boundary of an open Lipschitz set $\omega\subset\Omega$ in a bounded computational domain $\Omega\subset\mathbb{R}^{d}$, with $d=2$ or $d=3$. Then, the signed distance function $\mathrm{sgndist}(\cdot,\mathcal{S})\colon\Omega\rightarrow\mathbb{R}$ of $\mathcal{S}$ is defined as

$$
\displaystyle\mathrm{sgndist}(x,\mathcal{S})\coloneqq\begin{cases}\mathrm{dist}(x,\mathcal{S})\quad&\text{, if }x\notin\omega\\
-\mathrm{dist}(x,\mathcal{S})\quad&\text{, if }x\in\omega\,.\end{cases}
$$

The SDF of $\mathcal{S}$ is known to solve the eikonal equation as a function $\phi:\Omega\to\mathbb{R}$ with $\phi=0$ on $\mathcal{S}$, and

$$
\displaystyle\|\nabla\phi\|=1\quad\text{a.e. in }\Omega.
$$

The set of these solutions is very large in general. The concept of *viscosity solutions* allows to identify the signed distance function as the unique weak solution $\phi$ of (1) with $\phi=0$ on $\mathcal{S}$ via a comparison principle, see fig.˜3 for a 1D and a 2D example. This notion implies that the SDF is maximal in absolute values, compared to other solutions of (1). Functions solving the eikonal equation possess an additional structural property. In the direction normal to its level sets $\phi$ increases linearly, i.e.

$$
\displaystyle D^{2}\phi\,\nabla\phi=0
$$

in regions where $\phi$ is twice differentiable. This directly follows from a differentiation of $\|\nabla\phi\|^{2}=1$ and using the symmetry of $D^{2}\phi$. Another, geometric interpretation of (2) is that the gradient vector field $\nabla\phi$ of the signed distance $\phi$ is constant on rays pointing in the gradient direction, i.e.

$$
0=\partial_{t}\left(\nabla\phi(x+t\nabla\phi(x))\right)|_{t=0}=D^{2}\phi(x)\nabla\phi(x)\,.
$$

A key insight is that, while functions $\phi$ solving (1) are Lipschitz and differentiable almost everywhere, their gradient jumps on a set $J_{\nabla\phi}$, cf. fig.˜3. In fact, even for the SDF of a smooth surface $\mathcal{S}$, the jump set of the gradient is a lower dimensional set of those points where the nearest-point projection onto $\mathcal{S}$ is not defined, also denoted as the medial axis of $\mathcal{S}$. In [^33] and [^31], the degeneracy condition of the Hessian (2) was used in the near-field of the surface to learn signed distance functions. Different from these approaches, our method takes advantage of (2) globally and at the same time controls the co-dimension $1$ area of the jump set $J_{\nabla\phi}$.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/plot_1d_square.jpg)

Figure 3: Left: two graphs of one-dimensional functions satisfying the eikonal equation a.e., with zero boundary conditions at the green dots. The measure of the jump set (red dots) of the gradient J ∇ ϕ J\_{\\nabla\\phi} is 3 at the top, and 1 for the viscosity solution displayed at the bottom. Right: SDF of a square (zero-level set in green) with the jump set (in red), also denoted as the medial axis. Note that the equally spaced levelsets indicate the constant slope of the SDF in normal direction.

## 3 Method

We introduce a variational approach to compute the global SDF of a surface and its medial axis, i.e., the discontinuity set of the SDF’s gradient. The cost functional measures the second order derivative in normal direction where the SDF is smooth. Furthermore, it controls the area measure of the discontinuity set.

### 3.1 Higher-order variational problem with jump term

In what follows, we describe the SDF as the minimizer of a variational problem that picks up the observations made in section˜2 in terms of cost functional components which

1. penalize deviations from the eikonal equation (1),
2. regularize with a higher order term by enforcing the characteristic linearity along gradient directions away from the jump set $J_{\nabla\phi}$, cf. (2), and
3. control the measure of $J_{\nabla\phi}$ where $\nabla\phi$ is discontinuous.

To this end, we take into account

$$
\int_{\Omega}\left|\|\nabla\phi\|^{2}-1\right|^{2}\textrm{d}x
$$

as a penalty functional for the eikonal equation (1) and measure as a regularizer the second order derivative in normal direction in the square $L^{2}$ norm

$$
\int_{\Omega\setminus J_{\nabla\phi}}\|D^{2}\phi\,\nabla\phi\|^{2}\,\mathrm{d}x\,,
$$

on the complement of $J_{\nabla\phi}$. Furthermore, we include the $(d-1)$ -dimensional Hausdorff measure

$$
\mathcal{H}^{d-1}(J_{\nabla\phi})\,,
$$

of the gradient jump set.

The functional defined as a weighted sum of these three cost components is a higher-order analogue of the Mumford-Shah energy for image segmentation, cf. [^28]. The term $\mathcal{H}^{d-1}(J_{\nabla\phi})$ corresponds to the discontinuity penalty for the jump of the image intensity, where we here evaluate the jump of the *gradient* of $\phi$. The term penalizing the second order derivative in normal direction replaces the Dirichlet energy of the image intensity. In both cases integration is performed only on the complement of the jump set. Finally, the eikonal loss plays the role of the fidelity term measuring the discrepancy of the denoised image from the noisy input.

Let us emphasize that on the one hand the eikonal loss is a non-convex functional of $\nabla\phi$, where on the other hand the highest order term is convex in the Hessian $D^{2}\phi$.

The functional with the above three components does not necessarily recover the *viscosity* solution. In fact, there exist a.e. solutions of the Eikonal equation with smaller jump set measure than the viscosity solution as depicted in fig.˜4.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/AV_Giga.jpg)

Figure 4: Isolines of different solutions to the eikonal equation vanishing on the surface contour (in green) together with the gradient jump set (in red). Left: viscosity solution, Right: solution with shorter length of J ∇ ϕ J\_{\\nabla\\phi} (example from 4 ).

To increase the preference for the maximality property of the viscosity solution we add a further penalty

$$
\mathcal{L}_{\text{exp}}[\phi]\coloneqq\sum_{p=1}^{3}\int_{\Omega}\exp(-\alpha_{p}|\phi|^{p})\,\mathrm{d}x\,,
$$

favouring high values of $|\phi|$ for $\alpha_{p}>0$. A similar loss function was also used in [^30] and [^33]. Here, we consider $\alpha_{1}=100$, $\alpha_{2}=100$, and $\alpha_{3}=10$.

A minimizer of the resulting functional vanishing on $\mathcal{S}$ is expected to be a signed distance function, up to the overall sign. The identification of inside and outside is not required. In fact, the gradient of an unsigned distance function jumps on $\mathcal{S}$, which leads to an avoidable increase of $\mathcal{H}^{d-1}(J_{\nabla\phi})$.

### 3.2 Phase field approximation

A direct discretization of the medial axis, with its generally complex topology is a serious challenge. To handle such singularity sets phase field models provide a powerful alternative. Here, we pick up the phase field model introduced by Ambrosio and Tortorelli, cf. [^2], for the approximation of the Mumford-Shah energy. Our model differs from the two-phase Modica-Mortola phase field model used in [^22], which uses network based implicit surface representation and computes of a local approximation of the signed distance function in a postprocessing step. In our approach, we apply the Ambrosio-Tortorelli model to represent the medial axis of the surface $\mathcal{S}$ as an in general open discontinuity set of the SDF gradient.

At first, for a small parameter $\varepsilon>0$, we introduce the scaled eikonal loss

$$
\displaystyle\mathcal{L}_{\text{eik}}^{\varepsilon}[\phi]\coloneqq\frac{1}{\varepsilon}\int_{\Omega}\left|\|\nabla\phi\|^{2}-1\right|^{2}\textrm{d}x\,,
$$

which will imply the convergence of $\phi$ with bounded loss to a solution of (1) for $\varepsilon$ tending to $0$. Then, we introduce the higher order regularization functional

$$
\displaystyle\mathcal{L}_{\text{HO}}^{\varepsilon}[\phi,v]\coloneqq\int_{\Omega}v^{2}\|D^{2}\phi\nabla\phi\|^{2}+\varepsilon^{2}\|D^{2}\phi\|^{2}\textrm{d}x
$$

together with the Ambrosio-Tortorelli loss functional

$$
\displaystyle\mathcal{L}_{\text{AT}}^{\varepsilon}[v]\coloneqq\int_{\Omega}\varepsilon\|\nabla v\|^{2}+\frac{1}{4\varepsilon}(v-1)^{2}\textrm{d}x\,,
$$

where $v:\Omega\to\mathbb{R}$ is a phase field function with $v\approx 1$ away from the jump set and decaying significantly in a transition region of width $\varepsilon$ around the jump set. For fixed $\varepsilon$ the first term in $\mathcal{L}_{\text{AT}}^{\varepsilon}$ fosters smoothness of the phase field, whereas the second term expresses a preference for $v\approx 1$ on sets of positive Lebesgue measure. For $\varepsilon\to 0$ and minimizing tuples $(\phi^{\varepsilon},v^{\varepsilon})$ the associated Ambrosio-Tortorelli loss $\mathcal{L}_{\text{AT}}^{\varepsilon}[v^{\varepsilon}]$ is expected to converge to the $\mathcal{H}^{d-1}$ -area of the gradient jump set of the limit of $\phi^{\varepsilon}$. Small values of $v^{\varepsilon}$ allow for local regularity defects in $D^{2}\phi^{\varepsilon}\nabla\phi^{\varepsilon}$ close to the jump set. In fact, the higher order regularization is active only in direction of the gradient of the SDF approximation and only away from the jump set. To render the phase field model for fixed $\varepsilon>0$ wellposed, we add the second term in $\mathcal{L}_{\text{HO}}^{\varepsilon}$ as an isotropic regularizer, uniformly weighted with $\varepsilon^{2}$.

Finally, we have to ensure for the SDF that $\phi\approx 0$ on $\mathcal{S}$. This is resembled by the reconstruction loss

$$
\displaystyle\mathcal{L}_{\text{recon}}^{\varepsilon}[\phi]\coloneqq\frac{1}{\varepsilon^{2}}\int_{\mathcal{S}}\phi^{2}\textrm{d}\mathcal{H}^{d-1}\,.
$$

Altogether, we obtain the total loss functional

$$
\displaystyle\mathcal{L}_{\text{total}}^{\varepsilon}[\phi,v]\coloneqq
$$
 
$$
\displaystyle\gamma_{\text{HO}}\mathcal{L}_{\text{HO}}^{\varepsilon}[\phi,v]+\gamma_{\text{AT}}\mathcal{L}_{\text{AT}}^{\varepsilon}[v]+\gamma_{\text{recon}}\mathcal{L}_{\text{recon}}^{\varepsilon}[\phi]
$$
 
$$
\displaystyle+\gamma_{\text{eik}}\mathcal{L}_{\text{eik}}^{\varepsilon}[\phi]+\gamma_{\text{exp}}\mathcal{L}_{\text{exp}}[\phi]\,,
$$

with positive weights $\gamma_{\text{HO}},\gamma_{\text{AT}},\gamma_{\mathrm{recon}},\gamma_{\mathrm{eik}},\gamma_{\text{exp}}>0$.

The following theorem establishes the existence of minimizing tuples $(\phi^{\varepsilon},v^{\varepsilon})$ for $\varepsilon>0$.

###### Theorem 3.1

Let the surface $\mathcal{S}$ be the boundary of an open Lipschitz set $\omega\subset\Omega$ in the bounded computational domain $\Omega\subset\mathbb{R}^{d}$, with $d=2$ or $d=3$. Then, for fixed $\varepsilon>0$, there exists a minimizing tuple $(\phi^{\varepsilon},v^{\varepsilon})\in H^{2}(\Omega)\times H^{1}(\Omega)$ of $\mathcal{L}_{\text{total}}^{\varepsilon}[\cdot,\cdot]\,$.

Here, $H^{s}(\Omega)$ is the space of functions with square integrable weak derivatives of order up to $s$. The proof can be found in appendix˜A.

###### Remark 3.1 (Recovery of the limit loss functionals)

For $\mathcal{S}$ being Lipschitz and a solution $\phi$ of (1) vanishing on $\mathcal{S}$, with sufficiently regular jump set $J_{\nabla\phi}$, we can construct an approximating sequence $(\phi^{\varepsilon})_{\varepsilon}\subset H^{2}(\Omega)$ with $\phi^{\varepsilon}\rightarrow\phi$ strongly in $H^{1}(\Omega)$ accompanied by a sequence of phase field functions $(v^{\varepsilon})_{\varepsilon}$ in $H^{1}(\Omega)$ such that the components of the phase field loss evaluated at $(\phi^{\varepsilon},v^{\varepsilon})$ recover the corresponding limit quantities, i.e.

$$
\displaystyle\lim_{\varepsilon\rightarrow 0}\mathcal{L}_{\text{HO}}^{\varepsilon}[\phi^{\varepsilon},v^{\varepsilon}]
$$
 
$$
\displaystyle=\int_{\Omega\setminus J_{\nabla\phi}}\left\|D^{2}\phi\,\nabla\phi\right\|^{2}\,\textrm{d}x=0\,,
$$
$$
\displaystyle\lim_{\varepsilon\rightarrow 0}\mathcal{L}_{\text{exp}}[\phi^{\varepsilon}]
$$
 
$$
\displaystyle=\mathcal{L}_{\text{exp}}[\phi]\,,
$$
$$
\displaystyle\lim_{\varepsilon\rightarrow 0}\mathcal{L}_{\text{AT}}^{\varepsilon}[v^{\varepsilon}]
$$
 
$$
\displaystyle=\mathcal{H}^{d-1}(J_{\nabla\phi})\,,
$$
$$
\displaystyle\lim_{\varepsilon\rightarrow 0}\mathcal{L}_{\text{eik}}^{\varepsilon}[\phi^{\varepsilon}]
$$
 
$$
\displaystyle=0\,,\quad\lim_{\varepsilon\rightarrow 0}\mathcal{L}_{\text{recon}}^{\varepsilon}[\phi^{\varepsilon}]=0\,.
$$

To see this, we construct $\phi^{\varepsilon}$ via a convolution of $\phi$ with the standard mollifier

$$
\displaystyle\rho(x)\coloneqq\begin{cases}C_{d}\exp(\frac{1}{\|x\|^{2}-1})\quad&\text{, if }\|x\|<1\\
0\quad&\text{, else}\end{cases},\quad\rho^{b_{\varepsilon}}(x)\coloneqq b_{\varepsilon}^{-d}\rho(\frac{x}{b_{\varepsilon}})\,,
$$

restricted to a tubular neighborhood $(J_{\nabla\phi})_{b_{\varepsilon}}$ of the jump set $J_{\nabla\phi}$ with radius $b_{\varepsilon}$, where $b_{\varepsilon}$ is a small parameter with $\tfrac{b_{\varepsilon}}{\varepsilon}\rightarrow 0$ and $\tfrac{\varepsilon^{2}}{b_{\varepsilon}}\rightarrow 0$. For the construction of $v^{\varepsilon}$ we follow [^2] and define

$$
\displaystyle v^{\varepsilon}(x)\coloneqq\begin{cases}0\quad&\text{in }(J_{\nabla\phi})_{b_{\varepsilon}},\\
1-\exp(\frac{b_{\varepsilon}-\mathrm{dist}(x,J_{\nabla\phi})}{2\varepsilon})&\text{else.}\end{cases}
$$

A sketch of $v^{\varepsilon}$ and $\phi^{\varepsilon}$ is displayed in fig.˜5.

For the phase field approximation of the Mumford-Shah model the construction of a recovery sequence can be complemented to a full $\Gamma$ -convergence result (cf. [^2]). For our model, the associated $\liminf$ inequality relating the phase field functional $\mathcal{L}_{\text{HO}}^{\varepsilon}$ and the limit functional $\int_{\Omega\setminus J_{\nabla\phi}}\left\|D^{2}\phi\,\nabla\phi\right\|^{2}\,\textrm{d}x$ as well as the recovery sequence for general eikonal solutions is still open.

Figure 5: On a slice in direction normal to $J_{\nabla\phi}$ (red) the constructed $\phi^{\varepsilon}$ and $v^{\varepsilon}$ are depicted for a given solution $\phi$ of the eikonal equation with $\phi=0$ on $\mathcal{S}$ (in green).

## 4 Implementation

Throughout all experiments, we consider surfaces $\mathcal{S}$, represented by unoriented point clouds contained in the computational domain $\Omega\coloneqq[-1.2,1.2]^{d}$. For the phase field parameter $\varepsilon$ we choose $\varepsilon=10^{-3}$ for $d=2$ and $\varepsilon=10^{-4}$ for $d=3$. We implement our method using the Pytorch framework, cf. [^29]. All experiments were performed on a workstation with two AMD EPYC 7402 processors, 256GB RAM, and an NVIDIA A100 with 40GB memory. We plan to release code upon publication.

### 4.1 Networks

To represent the SDF $\phi$ and the phase field $v$, we use two separate networks $\phi_{\theta}$ and $v_{\eta}$ with parameter vectors $\theta$ and $\eta$. The input to both networks is the spatial coordinate $x\in\Omega$.

For the phase field network $v_{\eta}$, we employ a ResNet architecture, cf. [^17], with sinusoidal activation functions, referred to as SIREN activations (cf. [^30]). To encourage the phase field to remain in $[0,1]$ without overshooting, we augment the final layer by concatenating the ResNet output with a sigmoid function $\tfrac{e^{\delta x}}{1+e^{\delta x}}$ with $\delta=0.1$, representing a smooth transitions from $0$ as $x\to-\infty$ to $1$ as $x\to\infty$. For 2D experiments, we use 4 ResNet blocks with 64 hidden units each, and for 3D, we use 6 ResNet blocks with 128 hidden units each.

For the SDF network we employ a quadratic neural network (QuaNet, cf. [^13]). Concretely, the SDF $\phi_{\theta}$ is parameterized by an MLP with 256 hidden units per layer, where we use 4 layers for 2D and 8 layers for the 3D experiments. Each hidden layer implements a quadratic expansion of its pre-activation followed by a SIREN activation.

We optimize using stochastic gradient descent simultaneously for the phase field and SDF network parameters using the Adam approach [^19] with different gradient descent parameters. Specifically, we use $\beta_{1}=0.9$ and $\beta_{2}=0.999$ for the phase field network, and $\beta_{1}=0.9$ and $\beta_{2}=0.98$ for the SDF network. The learning rates are set to $5\times 10^{-4}$ for the phase field network and $5\times 10^{-5}$ for the SDF network, and are decreased during training.

### 4.2 Adaptive sampling

To approximate the volume integrals in the loss functional, we employ a Monte Carlo quadrature scheme and use an adaptive sampling strategy during training of our 3D models. In particular, we sample more points close to the approximate medial axis and the surface with suitably adapted integration weights, to achieve a high resolution of the phase field close to the jump set $J_{\nabla\phi_{\theta}}$ and a good reconstruction of the surface $\mathcal{S}$. This overall results in an improved approximation of the global signed distance function. For this we utilize a simple and yet effective sampling strategy, based on local grid refinement, with refinement level $i\in\mathbb{N}$, in regions with $|\phi_{\theta}|<2^{-i}\tau_{\mathrm{sdf}}$ or $v_{\eta}<\tau_{\mathrm{pf}}$, where $\tau_{\mathrm{sdf}}$ and $\tau_{\mathrm{pf}}$ are given thresholds. In practice we use $\tau_{\mathrm{sdf}}=0.1$ and $\tau_{\mathrm{pf}}=0.75$. A sketch, displaying this adaptive sampling strategy, is shown in fig.˜6. The detailed algorithm for one training based on adaptive sampling is explained in algorithm˜1. The weights associated with the sample points in cells on refinement level $i$ are $(2^{i\,d}n)^{-1}h^{d}$. Here, $h>0$ is the initial grid size and $n\in\mathbb{N}$ is the number of sample points in the cell. The training of the parameter vectors $\theta$ and $\eta$ for the networks $\phi_{\theta}$ and $v_{\theta}$ is repeated until a stopping criterion is met. For our experiments, we trained for a total of 30 epochs.

Furthermore, to accurately approximate the reconstruction loss $\mathcal{L}_{\text{recon}}^{\varepsilon}$, we employ the local weighting scheme introduced in [^34].

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/adaptive_grid.jpg)

Figure 6: Left: regular grid, where cells with | ϕ θ < τ sdf |\\phi\_{\\theta}|<\\tau\_{\\mathrm{sdf}} or v η pf v\_{\\eta}<\\tau\_{\\mathrm{pf}} are marked. Right: Subdivision of marked cells and sampling of points on reference cell.

### 4.3 Loss term scheduling

For both 2D and 3D experiments, we employ a three-phase training schedule, where the weights in (3) are adapted during training. The starting and final weights are listed in appendix˜B and appendix˜C.

##### Phase 1 (SDF initialization).

We start training to obtain a proper first approximation of the SDF and some approximate reconstruction of the surface as its zero-level set. As an initialization, we choose an approximate SDF of the unit sphere, i.e. $\phi_{\theta}(x)\approx\|x\|-1$. During this phase, we set $\gamma_{\text{exp}}$ to a high value to encourage SDF growth away from the surface, while keeping $\gamma_{\text{eik}}$ and $\gamma_{\text{HO}}$ relatively low and thereby eliminating ghost geometry. The phase field $v\equiv 1$ is held fixed in this phase, and we solely train the SDF.

##### Phase 2 (joint SDF and phase field optimization).

After a fixed number of training epochs (in our implementation $5$), we activate the phase field optimization and stepwise increase $\gamma_{\text{eik}}$ and $\gamma_{\text{HO}}$ up to the destinated values. Simultaneously, we decrease $\gamma_{\text{exp}}$ down to the destinated value. The phase field is generally easier to train and we observe a relatively quick convergence.

##### Phase 3 (SDF refinement).

In the final phase (in our implementation after 20 epochs) we freeze the phase field network and continue optimizing the SDF. This allows for a fine-tuning of the distance function in light of the fixed phase field representing the discontinuity set while avoiding unnecessary computational overhead from further phase field updates. In this phase we additionally evaluate the second-order term $v_{\eta}^{2}\|D^{2}\phi_{\theta}\nabla\phi_{\theta}\|^{2}$ on the point cloud.

This scheduling strategy has shown to ensure stable convergence in application. It leverages the complementary roles of the eikonal constraint, higher-order regularization, and phase field approximation throughout the training process.

## 5 Experiments

### 5.1 Experiments in 2D

We test our method on the two-dimensional dataset provided by [^32]. The computed SDFs and phase fields corresponding to this dataset are shown in fig.˜15 in appendix˜B, together with the ground truth solution for a subset, showing good agreement of our SDF solution, and of the medial axis approximated by the phase field.

In fig.˜7, we display a visual comparison of the computed SDF of four of the 2D-shapes of the Hotspot method [^32] and our method, compared to the ground truth. Especially inside the shapes, our method shows better distance computation visually, compared to Hotspot. The deviation from the ground truth is localized at very few spots such as the door of the house contour on the right.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/2D_comparison.jpg)

Figure 7: From top to bottom: Ground truth SDF solution, Computed SDF using the Hotspot method 32, Computed SDF using our method, Computed SDF using our method with the 0.25 -sublevel set of the phase field (PF, in red).

### 5.2 Experiments in 3D

#### 5.2.1 Evaluation metrics

Both high-detail surface reconstruction and accurate distance estimation are crucial for a good SDF approximation. We employ different metrics to evaluate the performance of our method with respect to these two aspects.  
First, to measure the surface reconstruction error, we compute the Chamfer distance

$$
\mathbf{d_{C}}\coloneqq\frac{1}{|\mathcal{P}|}\sum_{x\in\mathcal{P}}\min_{y\in\hat{\mathcal{P}}}\|x-y\|+\frac{1}{|\hat{\mathcal{P}}|}\sum_{y\in\hat{\mathcal{P}}}\min_{x\in\mathcal{P}}\|x-y\|,
$$

Additionally, we compute the Hausdorff distance

$$
\mathbf{d_{H}}\coloneqq\max\left\{\max_{x\in\mathcal{P}}\min_{y\in\hat{\mathcal{P}}}\|x-y\|,\max_{y\in\hat{\mathcal{P}}}\min_{x\in\mathcal{P}}\|x-y\|\right\}\,.
$$

For both Hausdorff and Chamfer distance, we use two point clouds $\mathcal{P}$ and $\hat{\mathcal{P}}$ with $100$ k points each, sampled from the ground truth surface $\mathcal{S}$ and the zero-level set $\hat{\mathcal{S}}$ of the computed SDF, respectively. To extract the zero-level set of the computed SDF, we use Marching Cubes with high resolution (in our implementation $512^{3}$). Further, we evaluate the normal alignment error on the input surface by computing

$$
\mathbf{E_{n}}\coloneqq 1-\frac{1}{|\mathcal{F}|}\sum_{F\in\mathcal{F}}n(x_{F})\cdot\frac{\nabla\phi(x_{F})}{\|\nabla\phi(x_{F})\|}\,,
$$

where $n(x_{F})$ denotes the discrete normal of the respective ground truth mesh at the triangle center $x_{F}$ of the triangle $F\in\mathcal{F}$, with $\mathcal{F}$ denoting the set of all triangles, and $|\mathcal{F}|$ the number of triangles, respectively. To this end, we exploit that all considered unoriented point clouds come with ground truth meshes.

We further evaluate the quality of the results inside the whole computational domain $\Omega=[-1.2,1.2]^{3}$ using a point cloud $\mathcal{P}_{\Omega}$ of $50$ k uniformly sampled points and on a point cloud $\mathcal{P}_{\mathcal{N}}$ of $10$ k points in a narrow band $\mathcal{N}$ with distances in $[-0.1,0.1]$ from the ground truth surface $\mathcal{S}$. The set $\mathcal{N}$ was generated once per model using rejection sampling based on a mesh-based pointwise distance computation with high accuracy [^20] and was subsequently used consistently across all evaluations. This method also serves as a ground truth signed distance $\mathrm{sgndist}(\cdot,\mathcal{S})$ to the mesh surface $\mathcal{S}$, which we use for comparison to calculate the SDF error on the whole domain $\Omega$ and the narrow band $\mathcal{N}$, respectively:

$$
\displaystyle\mathbf{E}_{\mathbf{SDF}}^{\Omega}
$$
 
$$
\displaystyle\coloneqq\sqrt{\frac{1}{|\mathcal{P}_{\Omega}|}\sum_{x\in\mathcal{P}_{\Omega}}|\phi(x)-\mathrm{sgndist}(x,\mathcal{S})|^{2}}\,,
$$
$$
\displaystyle\mathbf{E}_{\mathbf{SDF}}^{\mathcal{N}}
$$
 
$$
\displaystyle\coloneqq\sqrt{\frac{1}{|\mathcal{P}_{\mathcal{N}}|}\sum_{x\in\mathcal{P}_{\mathcal{N}}}|\phi(x)-\mathrm{sgndist}(x,\mathcal{S})|^{2}}\,,
$$

as well as the eikonal error:

$$
\displaystyle\mathbf{E}_{\mathbf{eik}}^{\Omega}
$$
 
$$
\displaystyle\coloneqq\frac{1}{|\mathcal{P}_{\Omega}|}\sum_{x\in\mathcal{P}_{\Omega}}\left|1-\|\nabla\phi(x)\|\right|\,,
$$
$$
\displaystyle\mathbf{E}_{\mathbf{eik}}^{\mathcal{N}}
$$
 
$$
\displaystyle\coloneqq\frac{1}{|\mathcal{P}_{\mathcal{N}}|}\sum_{x\in\mathcal{P}_{\mathcal{N}}}|1-\|\nabla\phi(x)\||\,.
$$

#### 5.2.2 Comparison

For the three-dimensional case, we compare our method against 5 state-of-the-art methods for neural SDF computation, including the neural network based methods HeatSDF [^34], Hessian [^33], HotSpot [^32], and 1-Lip [^8]. For the HeatSDF method, we used the version of the code that uses Winding numbers for inside/outside segmentation.  
Furthermore, we compare to a grid-based method, the generalized signed distance method (GSD) [^14]. All experiments with GSD were carried out with a grid size of $128^{3}$ ($\approx$ 2 million nodes). For comparison, our method uses a combined number of degrees of freedom for SDF and phase field of about $85^{3}$ ($\approx 600\mathrm{k}$).  
Note, that 1-Lip, HeatSDF and GSD require oriented point clouds as input.

We first evaluate all methods on the surface-reconstruction-benchmark dataset (SRB, cf. [^6]), see fig.˜10, table˜1 and table˜2. Our method achieves high accuracy in regions with fine detail, while also reconstructing flat regions with almost no perturbation, and properly resembling sharp edges with jumping normal. Quantitatively, our approach ranks consistently among the top methods for both surface reconstruction (table˜1) and SDF accuracy metrics (table˜2).

<table><thead><tr><th></th><th colspan="2"><math><semantics><msub><mi>𝐝</mi> <mi>𝐂</mi></msub> <annotation>\mathbf{d_{C}}</annotation></semantics></math></th><th colspan="2"><math><semantics><msub><mi>𝐝</mi> <mi>𝐇</mi></msub> <annotation>\mathbf{d_{H}}</annotation></semantics></math></th><th colspan="2"><math><semantics><msub><mi>𝐄</mi> <mi>𝐧</mi></msub> <annotation>\mathbf{E_{n}}</annotation></semantics></math></th></tr><tr><th>Method</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th></tr></thead><tbody><tr><th>GSD <math><semantics><mrow><mo>(</mo><mo>⋆</mo><mo>)</mo></mrow> <annotation>(\star)</annotation></semantics></math></th><td>0.0302</td><td>0.0074</td><td>0.1171</td><td>0.0450</td><td>0.1249</td><td>0.1736</td></tr><tr><th>1-Lip</th><td>0.0573</td><td>0.0170</td><td>0.2468</td><td>0.0965</td><td>0.1188</td><td>0.0332</td></tr><tr><th>HeatSDF</th><td>0.0147</td><td>0.0030</td><td>0.1521</td><td>0.0609</td><td>0.0378</td><td>0.0255</td></tr><tr><th>Hessian</th><td>0.0102</td><td>0.0016</td><td>0.0521</td><td>0.0327</td><td>0.0116</td><td>0.0126</td></tr><tr><th>HotSpot</th><td>0.0105</td><td>0.0017</td><td>0.0704</td><td>0.0427</td><td>0.0114</td><td>0.0079</td></tr><tr><th>Ours</th><td>0.0104</td><td>0.0014</td><td>0.0645</td><td>0.0329</td><td>0.0110</td><td>0.0088</td></tr></tbody></table>

Table 1: Averaged surface reconstruction metrics for the SRB dataset. Mean and standard deviation are reported across all shapes. The best results are highlighted in bold and the second-best results are underlined. $(\star)$ denotes a grid based method.

In terms of computational efficiency, our method is competitive with existing approaches, though it simultaneously optimizes both the SDF and the medial axis approximation via the phase field. Further details on runtime comparisons are provided in table˜2.

Additionally, we compare results for a total of 40 shapes from the Thingy10k dataset [^37], where 20 shapes are randomly sampled from the full dataset and 20 shapes are sampled from the pool of shapes with the tags ’sculpture’ or ’scan’. The resulting selection of shapes is especially challenging for SDF reconstruction, including shapes with thin structures, high curvature, and flat regions. In fig.˜16 in appendix˜C qualitative results for a subset of these shapes is presented. Our method achieves high-quality surface reconstruction and accurate signed distance estimation across both local and global error metrics. Corresponding quantitative results are displayed in table˜3.

#### 5.2.3 Ablation

The combination of both SDF and phase field in the variational approach in (3) is crucial for the performance of our method. In particular, using the eikonal loss in combination with higher order regularization without a phase field is not sufficient to recover high quality SDFs (see fig.˜9).

#### 5.2.4 Sphere tracing

We demonstrate the practical utility of our SDF method in the context of sphere tracing, see [^16]. To this end, we employ the implementation provided by [^32]. In fig.˜11 we show rendering results obtained via sphere tracing, which computes surface intersections along rays using the signed distance values, rather than relying on intermediate mesh extraction methods such as marching cubes. These results highlight both the computational efficiency and the visual quality of our approach compared to existing methods.

## 6 Discussion

We propose a novel, second-order variational model to compute the SDF of a surface point cloud. The simultaneously optimized phase field represents the medial axis, i.e. the jump set $J_{\nabla\phi}$ of the gradient of the SDF, which promotes an accurate SDF identification. The method combines an eikonal loss with a second-order loss term enforcing the characteristic linearity of the SDF along gradient directions away from the medial axis, and a loss controlling the ($d-1$)-dimensional measure of $J_{\nabla\phi}$.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/errors_vs_dists.jpg)

Figure 8: Quantitative evaluation of the Eikonal error (top) and the SDF error (bottom) averaged across the five shapes from the SRB dataset and evaluated on ground truth distance bands of width 0.05 to the surface (horizontal axes). Triangles denote the maximum error on the respective narrow band. ( ⋆ ) (\\star) denotes a grid based method.

<table><thead><tr><th></th><th colspan="2"><math><semantics><msubsup><mi>𝐄</mi> <mi>𝐞𝐢𝐤</mi> <mi>Ω</mi></msubsup> <annotation>\mathbf{E}_{\mathbf{eik}}^{\Omega}</annotation></semantics></math></th><th colspan="2"><math><semantics><msubsup><mi>𝐄</mi> <mi>𝐞𝐢𝐤</mi> <mi>𝒩</mi></msubsup> <annotation>\mathbf{E}_{\mathbf{eik}}^{\mathcal{N}}</annotation></semantics></math></th><th colspan="2"><math><semantics><msubsup><mi>𝐄</mi> <mi>𝐒𝐃𝐅</mi> <mi>Ω</mi></msubsup> <annotation>\mathbf{E}_{\mathbf{SDF}}^{\Omega}</annotation></semantics></math></th><th colspan="2"><math><semantics><msubsup><mi>𝐄</mi> <mi>𝐒𝐃𝐅</mi> <mi>𝒩</mi></msubsup> <annotation>\mathbf{E}_{\mathbf{SDF}}^{\mathcal{N}}</annotation></semantics></math></th><th>Timings</th></tr><tr><th>Method</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th><th>avg. / shape</th></tr></thead><tbody><tr><th>GSD <math><semantics><mrow><mo>(</mo><mo>⋆</mo><mo>)</mo></mrow> <annotation>(\star)</annotation></semantics></math></th><td>0.0375</td><td>0.0048</td><td>0.0673</td><td>0.0292</td><td>0.0321</td><td>0.0060</td><td>0.0772</td><td>0.0083</td><td>11 mins.</td></tr><tr><th>1-Lip</th><td>0.0429</td><td>0.0106</td><td>0.0553</td><td>0.0276</td><td>0.4806</td><td>0.1063</td><td>0.0917</td><td>0.0051</td><td>74 mins.</td></tr><tr><th>HeatSDF</th><td>0.2228</td><td>0.0654</td><td>0.14313</td><td>0.0366</td><td>0.1431</td><td>0.0358</td><td>0.0134</td><td>0.0038</td><td>32 mins.</td></tr><tr><th>Hessian</th><td>0.7693</td><td>0.0628</td><td>0.6389</td><td>0.0639</td><td>0.4945</td><td>0.0842</td><td>0.0283</td><td>0.0054</td><td>19 mins.</td></tr><tr><th>HotSpot</th><td>0.0905</td><td>0.0637</td><td>0.1415</td><td>0.0523</td><td>0.4261</td><td>0.0412</td><td>0.0712</td><td>0.0082</td><td>  6 mins.</td></tr><tr><th>Ours</th><td>0.0106</td><td>0.0041</td><td>0.0631</td><td>0.0123</td><td>0.0406</td><td>0.0056</td><td>0.0103</td><td>0.0023</td><td>11 mins.</td></tr></tbody></table>

Table 2: Left: SDF evaluation metrics for the SRB dataset. Mean and standard deviation are reported across all shapes. The best results are highlighted in bold and the second-best results are underlined. $(\star)$ denotes a grid based method. Right: Average timings per shape for the different methods.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/SRB.jpg)

Ground truth

The simultaneous optimization of the SDF $\phi$ and the phase field $v$ combines the identification of smooth regions of the SDF and the medial axis (the jump set $J_{\nabla\phi}$). This in particular implies a detailed surface reconstruction and at the same time a reliable SDF in the far field. Our approach computes SDFs which are close to the groundtruth viscosity solution of the eikonal equation. Across various experiments we show that our method achieves state-of-the-art surface reconstruction and signed distance accuracy, while most other methods focus on either of those. Although our method involves two neural networks, together with second derivatives in the loss functional, we achieve runtimes comparable to those methods with only one neural network for the SDF. We do not require oriented point clouds or a segmentation of inside and outside, because minimizers of the loss $\mathcal{L}_{\text{total}}^{\varepsilon}$ are already approximations of the signed distance, up to a multiplication with $-1$.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/spheretracing.jpg)

Figure 11: Sphere tracing rendering results (in black and white) on a torus with rectangular cross section and a hand shape, for our method (bottom row), HotSpot (middle row), and Hessian (top row). The color plots show the required number of iterations per pixel. These experiments where conducted using the implementation from 32.

First-order loss functionals, which involve the non-convex eikonal loss suffer from a lack of lower semi-continuity. In contrast, our second-order variational phase field functional is well-posed, in particular the existence of minimizers of the phase field loss functional $\mathcal{L}_{\text{total}}^{\varepsilon}$ is ensured.

The behaviour of minimizers for $\varepsilon\to 0$ poses some analytical challenges. The co-dimension $2$ components of the medial axis are not penalized by the Ambrosio-Tortorelli energy approximating the $(d-1)$ -dimensional measure of $J_{\nabla\phi}$ (cf. fig.˜12).

![Refer to caption](https://arxiv.org/html/2604.16512v1/x1.jpg)

Figure 12: Left: level sets of the neural SDF ϕ θ \\phi\_{\\theta} of a torus with rectangular cross section. Middle: zero-level set of. Right: phase field approximation of J ∇ J\_{\\nabla\\phi}. In addition to the co-dimension 1 interior segments of the jump set, the phase field depicts a segment of the torus’s rotational axis, a co-dimension 2 component of.

The $\mathcal{L}_{\text{exp}}$ penalty fosters that minimizers $\phi$ are viscosity solutions, but there is so far no analytical guarantee.

Surfaces with very complex topology have an even more complex medial axis. For the fixed set of parameters used in this paper, the resulting phase field does not approximate the medial axis, leading to a severe mismatch in the SDF, cf. fig.˜13. In fact, our model would require a significantly smaller phase field parameter $\varepsilon$ and larger $\alpha_{p}$ in $\mathcal{L}_{\text{exp}}$, and correspondingly an increased network depth and width for an accurate approximation. Currently, this still fails to converge. Here, an adaptation of the optimization strategy is required.

![Refer to caption](https://arxiv.org/html/2604.16512v1/x2.jpg)

Ground truth

We believe that our second-order loss, in combination with the Ambrosio–Tortorelli functional, can be integrated into existing SDF methods to improve both surface reconstruction quality and the accuracy of the signed distance approximation. As a proof of concept, we introduce a variation of the two-step first order HeatSDF method, in which the original formulation of the second step loss functional is augmented with an additional higher order phase field regularization term during training, i.e.,

$$
\mathcal{L}_{\text{Modified}}[\phi,v]:=\mathcal{L}_{\text{HeatSDF}}^{\text{2nd}}[\phi]+\mathcal{L}_{\text{HO}}^{\varepsilon}[\phi,v]+\mathcal{L}_{\text{AT}}^{\varepsilon}[v]\,.
$$

This modified approach leads to a sharper and more consistent SDF estimation, compared to the original HeatSDF method (see fig.˜14). We chose the HeatSDF method for this proof of concept, since it is a first-order method that does not involve solving the Eikonal equation, and thus is inherently different to our approach.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/HeatSDF_w_PF.jpg)

Figure 14: HeatSDF method (left) and modified HeatSDF method with higher-order loss functional and phase field regularization (right). Top row shows improved level set quality near the surface (right) compared to the original approach (left) for a slice ( y = 0 y=0 ) of the 3D anchor shape from the SRB data set 6. The bottom row displays renderings of both results, where the right rendering (modified HeatSDF) exhibits reduced surface noise compared to the left (original HeatSDF).

## References

## Appendix A Proof of

For every $(\phi,v)\in H^{2}(\Omega)\times H^{1}(\Omega)$, the total loss $\mathcal{L}_{\text{total}}^{\varepsilon}[\phi,v]$ is finite. Let $(\phi_{i}^{\varepsilon},v_{i}^{\varepsilon})_{i\in\mathbb{N}}\subset H^{2}(\Omega)\times H^{1}(\Omega)$ be a minimizing sequence, i.e.

$$
\inf_{(\phi,v)}\mathcal{L}_{\text{total}}^{\varepsilon}[\phi,v]=\lim\limits_{i\rightarrow\infty}\mathcal{L}_{\text{total}}^{\varepsilon}[\phi_{i}^{\varepsilon},v_{i}^{\varepsilon}]\leq C<\infty\,.
$$

Using this boundedness and the Poincaré-type estimate

$$
\|\phi\|^{2}_{L^{2}(\Omega)}\leq C(\|\nabla\phi\|^{2}_{L^{2}(\Omega)}+\|\phi\|^{2}_{L^{2}(\mathcal{S})})
$$

there exists a constant $C$, s.t.

$$
\|\phi_{i}^{\varepsilon}\|_{H^{2}(\Omega)}+\|v_{i}^{\varepsilon}\|_{H^{1}(\Omega)}\leq C\quad\forall i\in\mathbb{N}\,.
$$

By the reflexivity of the spaces $H^{2}(\Omega)$ and $H^{1}(\Omega)$, there exist $(\phi^{\varepsilon},v^{\varepsilon})\in H^{2}(\Omega)\times H^{1}(\Omega)$, such that $\phi_{i}^{\varepsilon}\rightharpoonup\phi^{\varepsilon}$ weakly in $H^{2}(\Omega)$ and $v_{i}^{\varepsilon}\rightharpoonup v^{\varepsilon}$ weakly in $H^{1}(\Omega)$ for a subsequence (not relabeled). By Rellich’s theorem, there exist further subsequences, such that $\phi_{i}^{\varepsilon}\rightarrow\phi^{\varepsilon}$ strongly in $H^{1}(\Omega)$, and $v_{i}^{\varepsilon}\rightarrow v^{\varepsilon}$ strongly in $L^{2}(\Omega)$. By the weak lower semi-continuity of the norm, and by dominated convergence, we hence have

$$
\liminf_{i\rightarrow\infty}\mathcal{L}_{\text{AT}}^{\varepsilon}[v_{i}^{\varepsilon}]\geq\mathcal{L}_{\text{AT}}^{\varepsilon}[v^{\varepsilon}]\,,
$$

and

$$
\displaystyle\lim_{i\rightarrow\infty}\left(\mathcal{L}_{\text{eik}}^{\varepsilon}[\phi_{i}^{\varepsilon}]+\mathcal{L}_{\text{exp}}[\phi_{i}^{\varepsilon}]+\mathcal{L}_{\mathcal{S}}^{\varepsilon}[\phi_{i}^{\varepsilon}]\right)
$$
 
$$
\displaystyle=\mathcal{L}_{\text{eik}}^{\varepsilon}[\phi^{\varepsilon}]+\mathcal{L}_{\text{exp}}[\phi^{\varepsilon}]+\mathcal{L}_{\mathcal{S}}^{\varepsilon}[\phi^{\varepsilon}]\,.
$$

Again, by weak lower semi-continuity of the norm, we have

$$
\liminf_{i\rightarrow\infty}\int_{\Omega}\varepsilon^{2}|D^{2}\phi_{i}^{\varepsilon}|^{2}\textrm{d}x\geq\int_{\Omega}\varepsilon^{2}|D^{2}\phi^{\varepsilon}|^{2}\textrm{d}x\,.
$$

Using again the boundedness of the total loss, the sequence $(v_{i}^{\varepsilon}D^{2}\phi_{i}^{\varepsilon}\nabla\phi_{i}^{\varepsilon})_{i\in\mathbb{N}}$ is bounded in $L^{2}(\Omega;\mathbb{R}^{d})$, hence there exists $\Theta^{\varepsilon}\in L^{2}(\Omega;\mathbb{R}^{d})$, such that

$$
v_{i}^{\varepsilon}D^{2}\phi_{i}^{\varepsilon}\nabla\phi_{i}^{\varepsilon}\rightharpoonup\Theta^{\varepsilon}\quad\text{weakly in }L^{2}(\Omega;\mathbb{R}^{d})
$$

for a subsequence. On the other hand, $(v_{i}^{\varepsilon})_{i\in\mathbb{N}}$ and $(\nabla\phi_{i}^{\varepsilon})_{i\in\mathbb{N}}$ are converging strongly in $L^{6}(\Omega)$ and $L^{6}(\Omega;\mathbb{R}^{d})$ for $d\in\{2,3\}$, see [^12]. Hence,

$$
v_{i}^{\varepsilon}D^{2}\phi_{i}^{\varepsilon}\nabla\phi_{i}^{\varepsilon}\rightharpoonup v^{\varepsilon}D^{2}\phi^{\varepsilon}\nabla\phi^{\varepsilon}\quad\text{weakly in }L^{\frac{6}{5}}(\Omega;\mathbb{R}^{d})\,.
$$

This implies $\Theta^{\varepsilon}=v^{\varepsilon}D^{2}\phi^{\varepsilon}\nabla\phi^{\varepsilon}$ almost everywhere and by the weak lower semicontinuity of the norm

$$
\liminf_{i\rightarrow\infty}\mathcal{L}_{\text{HO}}^{\varepsilon}[\phi_{i}^{\varepsilon},v_{i}^{\varepsilon}]\geq\mathcal{L}_{\text{HO}}^{\varepsilon}[\phi^{\varepsilon},v^{\varepsilon}]\,.
$$

Finally, we obtain

$$
\inf_{(\phi,v)}\mathcal{L}_{\text{total}}^{\varepsilon}[\phi,v]=\liminf_{i\rightarrow\infty}\mathcal{L}_{\text{total}}^{\varepsilon}[\phi_{i}^{\varepsilon},v_{i}^{\varepsilon}]\geq\mathcal{L}_{\text{total}}^{\varepsilon}[\phi^{\varepsilon},v^{\varepsilon}]\,,
$$

which implies that $(\phi^{\varepsilon},v^{\varepsilon})$ is a minimizer of the total loss $\mathcal{L}_{\text{total}}^{\varepsilon}$.

## Appendix B 2D Evaluation

For the experiments in 2D, we start the training with the weights

$$
\displaystyle(\gamma_{\text{HO}},\gamma_{\text{AT}},\gamma_{\mathrm{recon}},\gamma_{\mathrm{eik}},\gamma_{\text{exp}})_{\text{Phase 1}}=(10,0.2,10,0.1,100)\,.
$$

Following the schedule described in section˜4.3, we end with the final weights

$$
\displaystyle(\gamma_{\text{HO}},\gamma_{\text{AT}},\gamma_{\mathrm{recon}},\gamma_{\mathrm{eik}},\gamma_{\text{exp}})_{\text{Final}}=(10,0.2,10,0.1,1)\,.
$$

In fig.˜15, we provide additional qualitative results of the $2d$ -dataset from [^32], together with the phase field approximation of the medial axis.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/2D_full.jpg)

Figure 15: First three rows: Computed solutions for the 2D-dataset of 32. Zero-level set of SDF in green, 0.25 -sublevel set of phase field in red. Bottom row: Ground truth solution. Zero-level set of SDF in green, jump set J ∇ ϕ J\_{\\nabla\\phi} in red.

## Appendix C 3D Evaluation

For the experiments in 3D, we start the training with the weights

$$
\displaystyle(\gamma_{\text{HO}},\gamma_{\text{AT}},\gamma_{\mathrm{recon}},\gamma_{\mathrm{eik}},\gamma_{\text{exp}})_{\text{Phase 1}}=(1,0.02,0.01,0.05,500)\,.
$$

Following the schedule described in section˜4.3, we end with the final weights

$$
\displaystyle(\gamma_{\text{HO}},\gamma_{\text{AT}},\gamma_{\mathrm{recon}},\gamma_{\mathrm{eik}},\gamma_{\text{exp}})_{\text{Final}}=(2.5,0.2,0.5,0.2,200)\,.
$$

We provide additional experiments on a subset of the Thingi10k data set [^37]. In fig.˜16, a qualitative comparison of the zero-level set surface reconstruction on a selection of shapes is shown for different neural SDF methods.

For each shape, we extract 10k randomly sampled surface points for the training. For the evaluation we compare with the neural methods (Hessian, HotSpot, 1-Lip and HeatSDF). In table˜3, quantitative results evaluated on the whole subset of shapes are displayed. 1-Lip [^8] and HeatSDF [^34] did not converge on some of these shapes. We have excluded these shapes in the error evaluation of these methods.

<table><thead><tr><th></th><th colspan="2"><math><semantics><msub><mi>𝐝</mi> <mi>𝐂</mi></msub> <annotation>\mathbf{d_{C}}</annotation></semantics></math></th><th colspan="2"><math><semantics><msub><mi>𝐝</mi> <mi>𝐇</mi></msub> <annotation>\mathbf{d_{H}}</annotation></semantics></math></th><th colspan="2"><math><semantics><msub><mi>𝐄</mi> <mi>𝐧</mi></msub> <annotation>\mathbf{E_{n}}</annotation></semantics></math></th><th colspan="2"><math><semantics><msubsup><mi>𝐄</mi> <mi>𝐞𝐢𝐤</mi> <mi>Ω</mi></msubsup> <annotation>\mathbf{E}_{\mathbf{eik}}^{\Omega}</annotation></semantics></math></th><th colspan="2"><math><semantics><msubsup><mi>𝐄</mi> <mi>𝐞𝐢𝐤</mi> <mi>𝒩</mi></msubsup> <annotation>\mathbf{E}_{\mathbf{eik}}^{\mathcal{N}}</annotation></semantics></math></th><th colspan="2"><math><semantics><msubsup><mi>𝐄</mi> <mi>𝐒𝐃𝐅</mi> <mi>Ω</mi></msubsup> <annotation>\mathbf{E}_{\mathbf{SDF}}^{\Omega}</annotation></semantics></math></th><th colspan="2"><math><semantics><msubsup><mi>𝐄</mi> <mi>𝐒𝐃𝐅</mi> <mi>𝒩</mi></msubsup> <annotation>\mathbf{E}_{\mathbf{SDF}}^{\mathcal{N}}</annotation></semantics></math></th></tr><tr><th>Method</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th><th>mean</th><th>std.</th></tr></thead><tbody><tr><th>1-Lip</th><td>0.0536</td><td>0.0146</td><td>0.1576</td><td>0.0938</td><td>0.1413</td><td>0.1115</td><td>0.0728</td><td>0.0326</td><td>0.1109</td><td>0.0602</td><td>0.8607</td><td>0.1761</td><td>0.1109</td><td>0.0602</td></tr><tr><th>HeatSDF</th><td>0.0448</td><td>0.0208</td><td>0.1689</td><td>0.1206</td><td>0.0893</td><td>0.0947</td><td>0.2136</td><td>0.0209</td><td>0.1588</td><td>0.0724</td><td>0.3215</td><td>0.0942</td><td>0.0172</td><td>0.0103</td></tr><tr><th>Hessian</th><td>0.0110</td><td>0.0043</td><td>0.0472</td><td>0.0744</td><td>0.0753</td><td>0.1106</td><td>0.7615</td><td>0.0598</td><td>0.8185</td><td>0.2042</td><td>0.4841</td><td>0.1089</td><td>0.0318</td><td>0.0090</td></tr><tr><th>HotSpot</th><td>0.0130</td><td>0.0089</td><td>0.0566</td><td>0.0665</td><td>0.0818</td><td>0.1049</td><td>0.1069</td><td>0.0742</td><td>0.2143</td><td>0.0774</td><td>0.4266</td><td>0.0613</td><td>0.0671</td><td>0.0010</td></tr><tr><th>Ours</th><td>0.0124</td><td>0.0073</td><td>0.0847</td><td>0.1125</td><td>0.0741</td><td>0.1024</td><td>0.0769</td><td>0.0829</td><td>0.0189</td><td>0.0164</td><td>0.0253</td><td>0.0079</td><td>0.0871</td><td>0.0174</td></tr></tbody></table>

Table 3: Error analysis for the Thingy10k dataset. Mean and standard deviation are reported across all shapes. The best results are highlighted in bold and the second-best results are underlined.

![Refer to caption](https://arxiv.org/html/2604.16512v1/images/Thingy.jpg)

Figure 16: Marching Cubes results for the zero-level sets of the computed SDFs for a selection of shapes from the Thingy10k dataset. The first seven shapes are from those without tag specification. The last six shapes are with the tag ’sculpture’ or ’scan’. Runs that did not converge to an extractable geometry have been marked with an "X".

The experiments are conducted on a subset of models from the Thingi10K dataset. Specifically, we consider the following 40 models, identified by their Thing IDs: {1368069, 521602, 52073, 87522, 49420, 399567, 95778, 42025, 100679, 153957, 131602, 63497, 200688, 134622, 225950, 591249, 1452679, 384085, 814665, 370886, 353684, 64194, 84990, 61258, 73986, 133077, 133582, 91684, 68382, 131438, 65942, 896329, 84997, 73075, 101902, 103538, 354371, 39087, 55031, 133079}. Additionally, we used the model with Thing ID 16146, for our teaser figure. The hand geometry used for the spheretracing experiment is from [^36].  
The point cloud for the box (fig.˜2) and the torus with rectangular cross section (fig.˜12) were created by ourselbes.

[^1]: L. Ambrosio, C. De Lellis, and C. Mantegazza (1999-12) Line energies for gradient vector fields in the plane. Calculus of Variations and Partial Differential Equations 9, pp. 327–355. External Links: [Document](https://dx.doi.org/10.1007/s005260050144) Cited by: §1.

[^2]: L. Ambrosio and V. M. Tortorelli (1992) On the approximation of free discontinuity problems. Bollettino dell’Unione Matematica Italiana, Sezione B 6 (7), pp. 105–123. Cited by: §1, §3.2, §3.2, §3.2.

[^3]: P. Aviles and Y. Giga (1987) A mathematical problem related to the physical theory of liquid crystal configurations. Proc. Centre Math. Anal. Austral. Nat. Univ. 12, pp. 1–16. External Links: [Link](https://maths.anu.edu.au/files/CMAProcVol12-AvilesGiga_2.pdf) Cited by: §1.

[^4]: P. Aviles and Y. Giga (1996) The distance function and defect energy. Proceedings of the Royal Society of Edinburgh: Section A Mathematics 126 (5), pp. 923–938. External Links: [Document](https://dx.doi.org/10.1017/S0308210500023167) Cited by: §1, Figure 4.

[^5]: Y. Ben-Shabat, C. H. Koneputugodage, and S. Gould (2022) Digs: divergence guided shape implicit neural representation for unoriented point clouds. In Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition, New Orleans, USA, pp. 19323–19332. Cited by: §1, §1.

[^6]: M. Berger, J. A. Levine, L. G. Nonato, G. Taubin, and C. T. Silva (2013-04) A benchmark for surface reconstruction. ACM Trans. Graph. 32 (2). External Links: ISSN 0730-0301, [Document](https://dx.doi.org/10.1145/2451236.2451246) Cited by: §5.2.2, Figure 14.

[^7]: B. Bourdin, G.A. Francfort, and J-J. Marigo (2000) Numerical experiments in revisited brittle fracture. Journal of the Mechanics and Physics of Solids 48 (4), pp. 797–826. External Links: ISSN 0022-5096, [Document](https://dx.doi.org/https%3A//doi.org/10.1016/S0022-5096%2899%2900028-9), [Link](https://www.sciencedirect.com/science/article/pii/S0022509699000289) Cited by: §1.

[^8]: G. Coiffier and L. Béthune (2024) 1-Lipschitz neural distance fields. In Computer Graphics Forum, Vol. 43, pp. e15128. Cited by: Appendix C, §1, §5.2.2.

[^9]: K. Crane, C. Weischedel, and M. Wardetzky (2013) Geodesics in heat: a new approach to computing distance based on heat flow. ACM Transactions on Graphics (TOG) 32 (5), pp. 1–11. Cited by: §1.

[^10]: K. Crane, C. Weischedel, and M. Wardetzky (2017) The heat method for distance computation. Communications of the ACM 60 (11), pp. 90–99. Cited by: §1.

[^11]: A. Essakine, Y. Cheng, C. Cheng, L. Zhang, Z. Deng, L. Zhu, C. Schönlieb, and A. I. Aviles-Rivero (2025) Where do we stand with implicit neural representations? a technical and performance survey. Transactions on Machine Learning Research. Note: Survey Certification External Links: ISSN 2835-8856 Cited by: §1.

[^12]: L. C. Evans (1998) Partial differential equations. American Mathematical Society. Cited by: Appendix A.

[^13]: F. Fan, J. Xiong, and G. Wang (2020) Universal approximation with quadratic deep networks. Neural Networks 124, pp. 383–392. External Links: ISSN 0893-6080, [Document](https://dx.doi.org/https%3A//doi.org/10.1016/j.neunet.2020.01.007), [Link](https://www.sciencedirect.com/science/article/pii/S0893608020300095) Cited by: §4.1.

[^14]: N. Feng and K. Crane (2024-07) A heat method for generalized signed distance. ACM Trans. Graph. 43 (4). External Links: ISSN 0730-0301, [Link](https://doi.org/10.1145/3658220), [Document](https://dx.doi.org/10.1145/3658220) Cited by: §1, §5.2.2.

[^15]: G.A. Francfort and J.-J. Marigo (1998) Revisiting brittle fracture as an energy minimization problem. Journal of the Mechanics and Physics of Solids 46 (8), pp. 1319–1342. External Links: ISSN 0022-5096, [Document](https://dx.doi.org/https%3A//doi.org/10.1016/S0022-5096%2898%2900034-9), [Link](https://www.sciencedirect.com/science/article/pii/S0022509698000349) Cited by: §1.

[^16]: J. C. Hart (1996) Sphere tracing: a geometric method for the antialiased ray tracing of implicit surfaces. The Visual Computer 12, pp. 527–545. Cited by: §1, §5.2.4.

[^17]: K. He, X. Zhang, S. Ren, and J. Sun (2015) Deep residual learning for image recognition. 2016 IEEE Conference on Computer Vision and Pattern Recognition (CVPR), pp. 770–778. Cited by: §4.1.

[^18]: W. Jin and R. V. Kohn (2000) Singular perturbation and the energy of folds. Journal of Nonlinear Science 10 (3), pp. 355–390. External Links: [Document](https://dx.doi.org/10.1007/s003329910014), [Link](https://doi.org/10.1007/s003329910014) Cited by: §1.

[^19]: D. P. Kingma and J. Ba (2014) Adam: a method for stochastic optimization. CoRR abs/1412.6980. Cited by: §4.1.

[^20]: M. Kleineberg (2021) Mesh-to-sdf: calculate signed distance fields for arbitrary meshes. GitHub. External Links: [Link](https://github.com/marian42/mesh_to_sdf) Cited by: §5.2.1.

[^21]: C. D. Lellis (2002) An example in the gradient theory of phase transitions. ESAIM: Control, Optimisation and Calculus of Variations 7, pp. 285–289 (en). External Links: [Document](https://dx.doi.org/10.1051/cocv%3A2002012), [Link](https://www.numdam.org/articles/10.1051/cocv:2002012/), [MathReview Entry](https://www.ams.org/mathscinet-getitem?mr=1925030) Cited by: §1.

[^22]: Y. Lipman (2021) Phase transitions, distance functions, and implicit neural representations. In International Conference on Machine Learning, External Links: [Link](https://api.semanticscholar.org/CorpusID:235435938) Cited by: §1, §1, §3.2.

[^23]: P. Liu, Y. Zhang, H. Wang, M. K. Yip, E. S. Liu, and X. Jin (2024) Real-time collision detection between general SDFs. Computer Aided Geometric Design 111, pp. 102305. Cited by: §1.

[^24]: M. Manav, R. Molinaro, S. Mishra, and L. De Lorenzis (2024) Phase-field modeling of fracture with physics-informed deep learning. Computer Methods in Applied Mechanics and Engineering 429, pp. 117104. External Links: ISSN 0045-7825, [Document](https://dx.doi.org/https%3A//doi.org/10.1016/j.cma.2024.117104), [Link](https://www.sciencedirect.com/science/article/pii/S0045782524003608) Cited by: §1.

[^25]: Z. Marschner, S. Sellán, H. D. Liu, and A. Jacobson (2023) Constructive solid geometry on neural signed distance fields. In SIGGRAPH Asia 2023 conference papers, Sydney, Australia, pp. 1–12. Cited by: §1.

[^26]: I. Mehta, M. Chandraker, and R. Ramamoorthi (2022) A level set theory for neural implicit evolution under explicit flows. In European Conference on Computer Vision, Tel Aviv, Israel, pp. 711–729. Cited by: §1.

[^27]: L. Modica and S. Mortola (1977) Un esempio di $\Gamma$ -convergenza. Boll. Un. Mat. Ital. B (5) 14 (1), pp. 285–299. Cited by: §1, §1.

[^28]: D. Mumford and J. Shah (1989) Optimal approximations by piecewise smooth functions and associated variational problems. Comm. Pure Appl. Math. 42 (5), pp. 577–685. External Links: [Document](https://dx.doi.org/10.1002/cpa.3160420503) Cited by: §1, §3.1.

[^29]: A. Paszke, S. Gross, F. Massa, A. Lerer, J. Bradbury, G. Chanan, T. Killeen, Z. Lin, N. Gimelshein, L. Antiga, A. Desmaison, A. Köpf, E. Yang, Z. DeVito, M. Raison, A. Tejani, S. Chilamkurthy, B. Steiner, L. Fang, J. Bai, and S. Chintala (2019) PyTorch: an imperative style, high-performance deep learning library. In Proceedings of the 33rd International Conference on Neural Information Processing Systems, Cited by: §4.

[^30]: V. Sitzmann, J. N. P. Martel, A. W. Bergman, D. B. Lindell, and G. Wetzstein (2020) Implicit neural representations with periodic activation functions. External Links: 2006.09661, [Link](https://arxiv.org/abs/2006.09661) Cited by: §1, §1, §1, §3.1, §4.1.

[^31]: R. Wang, Z. Wang, Y. Zhang, S. Chen, S. Xin, C. Tu, and W. Wang (2023) Aligning gradient and hessian for neural signed distance function. In Advances in Neural Information Processing Systems, A. Oh, T. Naumann, A. Globerson, K. Saenko, M. Hardt, and S. Levine (Eds.), Vol. 36, pp. 63515–63528. External Links: [Link](https://proceedings.neurips.cc/paper_files/paper/2023/file/c87bd5843849884e9430f1693b018d71-Paper-Conference.pdf) Cited by: §1, §1, §2.

[^32]: Z. Wang, C. Wang, T. Yoshino, S. Tao, Z. Fu, and T. Li (2025) HotSpot: signed distance function optimization with an asymptotically sufficient condition. In CVPR, Cited by: Figure 15, Appendix B, §1, §1, Figure 7, §5.1, §5.1, §5.2.2, §5.2.4, Figure 11.

[^33]: Z. Wang, Y. Zhang, R. Xu, F. Zhang, P. Wang, S. Chen, S. Xin, W. Wang, and C. Tu (2023) Neural-singular-Hessian: implicit neural representation of unoriented point clouds by enforcing singular Hessian. ACM Transactions on Graphics (TOG) 42 (6), pp. 1–14. Cited by: §1, §1, §2, §3.1, §5.2.2.

[^34]: S. Weidemaier, F. Hartwig, J. Sassen, S. Conti, M. Ben-Chen, and M. Rumpf (2026) SDFs from unoriented point clouds using neural variational heat distances. Computer Graphics Forum n/a (n/a), pp. e70296. External Links: [Document](https://dx.doi.org/https%3A//doi.org/10.1111/cgf.70296), [Link](https://onlinelibrary.wiley.com/doi/abs/10.1111/cgf.70296), https://onlinelibrary.wiley.com/doi/pdf/10.1111/cgf.70296 Cited by: Appendix C, §1, §1, §1, §4.2, §5.2.2.

[^35]: H. Yang, Y. Sun, G. Sundaramoorthi, and A. Yezzi (2023) StEik: stabilizing the optimization of neural signed distance functions and finer shape representation. In Proceedings of the 37th International Conference on Neural Information Processing Systems, NIPS ’23, Red Hook, USA. Cited by: §1, §1.

[^36]: I. Yeh, C. Lin, O. Sorkine, and T. Lee (2010) Template-based 3d model fitting using dual-domain relaxation. IEEE Transactions on Visualization and Computer Graphics 17 (8), pp. 1178–1190. Cited by: Appendix C.

[^37]: Q. Zhou and A. Jacobson (2016) Thingi10K: a dataset of 10,000 3d-printing models. Preprint arXiv:1605.04797. Cited by: Appendix C, §5.2.2.