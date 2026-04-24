## Canonical Cube to Screen
- Defining the screen space
- Pixels' indices are in the form of (x, y), where both x and y are integers
- Pixels' indices are from (0,0) to (width - 1, height - 1)
- Pixel (x, y) is centered at (x + 0.5, y + 0.5)
- Transform in xy plane: $[-1, 1]^2$ to $[0,width] \times [0,height]$
- Viewport transform matrix
$$
M_{viewport} =
\begin{pmatrix}
\frac {width}2 & 0 & 0 & \frac {width}2 \\
0 & \frac {height}2 & 0 & \frac {height}2 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{pmatrix}
$$
## Triangles - Fundamental Shape Primitives
- Why?
	- Most basic polygon
	- Break up other polygons
- Unique properties
	- Guaranteed to be planar
	- Well-defined interior
	- Well-defined method for interpolating values at vertices over triangle (barycentric interpolation)
## Sampling Inside Function
- Axis-aligned bounding box (AABB)
```
for (int x = 0; x < xmax; ++x)
	output[x] = f(x);
```
- Sampling if each pixel center is inside triangle
```
for (int x = 0; x < xmax; ++x)
	for (int y = 0; y < ymax; ++y)
		image[x][y] = inside(tri, x + 0.5, y + 0.5);
```