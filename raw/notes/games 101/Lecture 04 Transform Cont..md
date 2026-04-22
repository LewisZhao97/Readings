$$
R_\theta = 
\begin{pmatrix}
cos\theta & -sin\theta \\
sin\theta & cos\theta
\end{pmatrix}
$$
$$
R_{-\theta} = 
\begin{pmatrix}
cos\theta & sin\theta \\
-sin\theta & cos\theta
\end{pmatrix} = R_\theta^T
$$
## (Orthogonal Matrix)
$$
R_{-\theta} = R_\theta^{-1} = R_\theta^T
$$
# 3D Transformation
## Rotation around x, y, z-axis
$$
R{_x}(\alpha) = 
\begin{pmatrix}
1 & 0 & 0 & 0 \\
0 & cos\theta & -sin\theta & 0 \\
0 & sin\theta & cos\theta & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}
$$
$$
R{_y}(\alpha) = 
\begin{pmatrix}
cos\theta & 0 & sin\theta & 0 \\
0 & 1 & 0 & 0 \\
-sin\theta & 0 & cos\theta & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}
$$
$$
R{_z}(\alpha) = 
\begin{pmatrix}
cos\theta & -sin\theta & 0 & 0 \\
sin\theta & cos\theta & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}
$$
## Compose any 3D rotation
$$
R_{xyz}(\alpha, \beta, \gamma) = R_x(\alpha)R_y(\beta)R_z(\gamma)
$$
# Viewing transformation
## View / Camera transformation
## Projection transformation
- Orthographic projection
- Perspective projection
## Orthographic Projection
- Camera located at origin, looking at -z
- Dorp z coordinate
- Translate and scale the resulting rectangle to $[-1, 1]^2$
### In general
- We create  a cuboid $[l,r][b,t][f,n]$ to the "canonical" cube $[-1,1]^3$
### Transformation Matrix
- Translate (**center** to origin) first, then scale (length / width / height to **2**)
$$
M_{ortho}=
\begin{bmatrix}
\frac 2 {r-l} & 0 & 0 & 0 \\
0 & \frac 2 {t-b} & 0 & 0 \\
0 & 0 & \frac 2 {n-f} & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
1 & 0 & 0 & -\frac {r+l} 2 \\
0 & 1 & 0 & -\frac {t+b} 2 \\
0 & 0 & 1 & -\frac {n+f} 2 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$
## Perspective Projection
- First "squish" the frustum into a cuboid (n -> n, f -> f)($M_{persp->orhtho}$)
- Orthographic projection
$$
y^{'} = \frac n z y
$$
$$
x^{'} = \frac n z x
$$
- In homogeneous coordinates
$$
M_{persp->ortho}^{(4 \times 4)}
\begin{pmatrix}
x \\ y \\ z \\ 1
\end{pmatrix} =>
\begin{pmatrix}
nx/z \\ ny/z \\ unknow \\ 1
\end{pmatrix} ==
\begin{pmatrix}
nx \\ ny \\ unknow \\ z
\end{pmatrix}
$$
- The part of matrix can be figured out
$$
M_{persp->ortho}^{(4 \times 4)} =
\begin{pmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
? & ? & ? & ? \\
0 & 0 & 1 & 0
\end{pmatrix}
$$
- Any point on the near plane will not change, the formula above can be replaced:
$$
\begin{pmatrix}
x \\ y \\ n \\ 1
\end{pmatrix} ==
\begin{pmatrix}
nx \\ ny \\ n^2 \\ n
\end{pmatrix}
$$
- So the third row must be
$$
\begin{pmatrix}
0 & 0 & A & B
\end{pmatrix}
\begin{pmatrix}
x \\ y \\ n \\ 1
\end{pmatrix} = n^2 ==> An + B = n^2
$$
- Any point on the far plane will also not change, so
$$
Af + B = f^2
$$
$$
A = n + f
$$
$$
B = -nf
$$
- Finally, the complete $M_{persp->ortho}$ goes to:
$$
M_{persp->ortho}^{(4 \times 4)} =
\begin{pmatrix}
n & 0 & 0 & 0 \\
0 & n & 0 & 0 \\
0 & 0 & n+f & -nf \\
0 & 0 & 1 & 0
\end{pmatrix}
$$