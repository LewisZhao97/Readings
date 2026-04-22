
> Note: A vector is a column vector by default

# Scale Matrix
$$
\begin{bmatrix}
x^{'} \\ y^{'}
\end{bmatrix} = 
\begin{bmatrix}
s & 0 \\
0 & s
\end{bmatrix}
\begin{bmatrix}
x \\ y
\end{bmatrix}
$$
## Scale (Non-Uniform)
$$
\begin{bmatrix}
x^{'} \\ y^{'}
\end{bmatrix} = 
\begin{bmatrix}
s_x & 0 \\
0 & s_y
\end{bmatrix}
\begin{bmatrix}
x \\ y
\end{bmatrix}
$$
## Reflection Matrix
$$
\begin{bmatrix}
x^{'} \\ y^{'}
\end{bmatrix} = 
\begin{bmatrix}
-1 & 0 \\
0 & 1
\end{bmatrix}
\begin{bmatrix}
x \\ y
\end{bmatrix}
$$
## Shear Matrix
$$
\begin{bmatrix}
x^{'} \\ y^{'}
\end{bmatrix} = 
\begin{bmatrix}
1 & a \\
0 & 1
\end{bmatrix}
\begin{bmatrix}
x \\ y
\end{bmatrix}
$$
# Rotate (about the origin(0,0), CCW by default)
## Rotate Matrix
$$
R_\theta = 
\begin{bmatrix}
cos\theta & -sin\theta\\
sin\theta & cos\theta
\end{bmatrix}
$$
# Translation
## Homogeneous Coordinates
- Add a third coordinate (w-coordinate)
	- 2D point = $(x,y,1)^T$
	- 2D vector= $(x,y,0)^T$
	- Vector will not change after translation, so w is 0
- Matrix representation of translations
$$
\begin{pmatrix}
x^{'} \\ y^{'} \\ z^{'} 
\end{pmatrix} =
\begin{pmatrix}
1 & 0 & t_x \\
0 & 1 & t_y \\
0 & 0 & 1
\end{pmatrix}
\begin{pmatrix}
x \\ y \\ 1
\end{pmatrix} =
\begin{pmatrix}
x + t_x \\ y + t_y \\ 1
\end{pmatrix}
$$
- Valid operation if w-coordinate of result is 1 or 0
	- vector + vector = vector
	- point - point = vector
	- point + vector = point
	- point + point = mid-point
$$
\begin{pmatrix}
x \\ y \\ w
\end{pmatrix} \text{ is the 2D point }
\begin{pmatrix}
x/w \\
y/w \\
1
\end{pmatrix}, \quad w \neq 0
$$
## Affine Transformations
- Affine map = linear map + translation
$$
\begin{pmatrix}
x^{'} \\
y^{'}
\end{pmatrix} =
\begin{pmatrix}
a & b \\
c & d
\end{pmatrix} \cdot
\begin{pmatrix}
x \\
y
\end{pmatrix} +
\begin{pmatrix}
t_x \\
t_y
\end{pmatrix}
$$
- Using homogeneous coordinates
$$
\begin{pmatrix}
x^{'} \\
y^{'} \\
1
\end{pmatrix} =
\begin{pmatrix}
a & b & t_x \\
c & d & t_y \\
0 & 0 & 1
\end{pmatrix} \cdot
\begin{pmatrix}
x \\
y \\
1
\end{pmatrix}
$$
## Inverse Transform
$$
M^{-1}
$$
# Composing Transform
- Matrix multiplication is **not** commutative
$$
R_{45} \cdot T_{(1,0)} \neq T_{(1,0)} \cdot R_{45}
$$
- Note that matrices are applied right to left
$$
T_{(1,0)} \cdot R_{45}
\begin{bmatrix}
x \\
y \\
1
\end{bmatrix} = 
\begin{bmatrix}
1 & 0 & 1 \\
0 & 1 & 0 \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
cos45^\circ & -sin45\circ & 0 \\
sin45\circ & cos45^\circ & 0 \\
0 & 0 & 1
\end{bmatrix}
\begin{bmatrix}
x \\
y \\
1
\end{bmatrix}
$$
- Sequence of affine transforms $A_1$ ,$A_2$ ,$A_3$ , ...
$$
A_n(...A_2(A_1(X))) = A_n... A_2 \cdot A_1 \cdot
\begin{pmatrix}
x \\
y \\
1
\end{pmatrix}
$$
- Pre-multiply n matrices to obtain a single matrix representing combined transform

# 3D Transformations
## Use homogeneous coordinates
- 3D point = $(x,y,z,1)^T$
- 3D vector = $(x,y,z,0)^T$

$$
\begin{pmatrix}
x \\ y \\ z \\ w
\end{pmatrix} \text{ is the 3D point }
\begin{pmatrix}
x/w \\
y/w \\
z/w \\
1
\end{pmatrix}, \quad w \neq 0
$$
## Use 4×4 matrices for affine transformations
$$\begin{pmatrix}
x^{'} \\
y^{'} \\
z^{'} \\
1
\end{pmatrix} =
\begin{pmatrix}
a & b & c & t_x \\
d & e & f & t_y \\
g & h & i & t_z \\
0 & 0 & 0 & 1
\end{pmatrix} \cdot
\begin{pmatrix}
x \\
y \\
z \\
1
\end{pmatrix}
$$