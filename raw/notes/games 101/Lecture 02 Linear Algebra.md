# Vectors and Matrices
## Vectors
- Direction
- Length
### Vectors Normalization
- A vector divided by its length
### Vectors Addition
- Parallelogram law & Triangle law
- Add coordinates
### Vectors Multiplication
- Dot product
	- Dot product in Cartesian Coordinates: Multiply every elements and Add
	- Dot product in Graphics: Find angle between two vectors， find projection of one on another
- Cross product
	- Cross product is orthogonal to two initial vectors
	- Direction determined by right-hand rule
	- Determine a point inside a triangle or not
## Matrices
### Matrix-Matrix Multiplication
- Non-commutative
	- AB and BA are different in general
- Associative and distributive
	- (AB)C = A(BC)
	- A(B+C) = AB + AC
	- (A+B)C = AC + BC
### Matrix-Vector Multiplication
- Treat vector as a column matrix(M × 1)
### Transpose of a Matrix
- Switch row and columns(ij - > ji)
$$
\begin{pmatrix}  
1 & 2 \\  
3 & 4 \\
5 & 6
\end{pmatrix} ^ T =
\begin{pmatrix}
1 & 3 & 5\\
2 & 4 & 6
\end{pmatrix}
$$

- Property
$$(AB)^T = B^TA^T$$

### Identity Matrix and Inverses

$$
I_3×3 = 
\begin{pmatrix}
1 & 0 & 0\\
0 & 1 & 0\\
0 & 0 & 1
\end{pmatrix}
$$
$$
AA^{-1} = A^{-1}A = I
$$
$$
(AB)^{-1} = B^{-1}A^{-1}
$$
### Vector multiplication in Matrix form
- Dot product?
$$
\vec{a}\cdot\vec{b} = \vec{a}^T\cdot\vec{b}
$$
$$=
\begin{pmatrix}
x_a & y_a & z_a
\end{pmatrix}
\begin{pmatrix}
x_b \\ y_b \\ z_b
\end{pmatrix}=
\begin{pmatrix}
x_a x_b + y_a y_b + z_a z_b
\end{pmatrix}
$$
- Cross product?
$$
\vec{a} \times \vec{b} = A^*\vec{b}
\begin{pmatrix}
0 & -z_a & y_a\\
z_a & 0 & -x_a\\
-y_a & x_a & 0
\end{pmatrix}
\begin{pmatrix}
x_b \\ y_b \\ z_b
\end{pmatrix}
$$
A* is dual matrix of vector a