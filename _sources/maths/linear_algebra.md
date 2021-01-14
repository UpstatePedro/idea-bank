# Linear algebra

## Tensors

- Scalars
- Vectors
- Matrices

## Scalars

Single numbers. eg. x = 1.0, y = 2.5

> Quantities defined solely by a magnitude

## Vectors & vector operations

A vector is a N-dimensional geometric entity, and a N x 1 data structure.

Vectors are used to represent a point in N-dimensional space.

#### Column vectors

All components of the vector are in a vertical column

$$
\mathbf{p} =
\begin{bmatrix}
p_{1} \\
p_{2} \\
p_{3} \\
\vdots \\
p_{N}
\end{bmatrix}
$$


> Notationally, column vectors can be written
$$
\mathbf{p} =
(
p_{1},
p_{2},
p_{3},
\dots,
p_{N}
)
$$
but this is NOT the same as a row vector!! **NB.** deliberate use of square brackets Vs parenthesis

#### Row vectors

All components of the vector are in a horizontal row.
Also the transpose of a column vector.

$$
\mathbf{p} =
\begin{bmatrix}
p_{1} &
p_{2} &
p_{3} &
\dots &
p_{N}
\end{bmatrix}
$$

For the following content, we start with the following setup:

```python
import numpy as np

p = np.array([ 1, 2, 3])
q = np.array([ 3, 2, 1])
```

### Vector addition / subtraction

- Two N-dimensional vectors can be added by performing the element-wise addition of their elements.
- The same is true of subtraction.
- Both operations yield another N-dimensional vector.

$$
\mathbf{p} + \mathbf{q} =
\begin{bmatrix}
p_{1} + q_{1} \\
p_{2} + q_{2} \\
p_{3} + q_{3} \\
\vdots \\
p_{N} + q_{N}
\end{bmatrix}
$$

$$
\mathbf{p} - \mathbf{q} =
\begin{bmatrix}
p_{1} - q_{1} \\
p_{2} - q_{2} \\
p_{3} - q_{3} \\
\vdots \\
p_{N} - q_{N}
\end{bmatrix}
$$

The vector $$\mathbf{d}$$ between two points $$\mathbf{a, b}$$ in N-dimensional space (from $$\mathbf{a}$$ to $$\mathbf{b}$$) is given by:
$$\mathbf{d = b - a}$$

```python
p + q

>> array([4, 4, 4])
```

### Scalar product

- Scalar multiplication of a vector multiplies each element of the vector by a scalar value.

$$
a\mathbf{p} = \mathbf{p}a =
\begin{bmatrix}
ap_{1} \\
ap_{2} \\
ap_{3} \\
\vdots \\
ap_{N}
\end{bmatrix}
$$

```python
p * 2

>> array([2, 4, 6])
```

### Linear Combination

- The combination of the vector addition and scalar product operations:

$$
a\mathbf{p} + b\mathbf{q} =

a\begin{bmatrix}
p_{1} \\
p_{2} \\
p_{3} \\
\vdots \\
p_{N}
\end{bmatrix}
+
b\begin{bmatrix}
q_{1} \\
q_{2} \\
q_{3} \\
\vdots \\
q_{N}
\end{bmatrix}

=
\begin{bmatrix}
ap_{1} + bq_{1} \\
ap_{2} + bq_{2}\\
ap_{3} + bq_{3}\\
\vdots \\
aq_{N} + bq_{N}
\end{bmatrix}
$$

```python
a = 2
b = 3
a*p + b*q

# (2 * [1, 2, 3]) + (3 * [3, 2, 1])
# [2, 4, 6] + [9, 6, 3]

>> array([11, 10, 9])
```

#### Results of linear combinations

##### 2D Space
Consider the possible results of $$a\mathbf{p} + b\mathbf{q}$$, where $$\mathbf{p}$$ & $$\mathbf{q}$$ are each **2-dimensional** vectors
- All possible values of $$a\mathbf{p}$$ lie on a single line (1 vector can create a 1D object in 2D space)
- Similarly, all possible values for $$b\mathbf{q}$$ lie on a single line
- **IF** $$\mathbf{q}$$ does not sit on the line $$a\mathbf{p}$$, then the possible combinations $$a\mathbf{p} + b\mathbf{q}$$ fill the entire 2D plane (2 2D vectors can create a 2D object to fill a 2D space)

##### 3D Space
If we then turn to the possible results of $$a\mathbf{p} + b\mathbf{q} + c\mathbf{r}$$, where $$\mathbf{p}$$ & $$\mathbf{q}$$ & $$\mathbf{r}$$ are each **3-dimensional** vectors
- All possible values of $$a\mathbf{p}$$ lie on a single line (1 vector can create a 1D object in 3D space). Similarly, all possible values for $$b\mathbf{q}$$ & $$c\mathbf{r}$$ each lie on a single line.
- The combination of any two of these 3D vectors can fill a 2D plane in the 3D space.
- **IF** none of the vectors share the same direction as another, then the possible combinations of all 3 3D vectors fills the entire 3D space (3 3D vectors can create a 3D object to fill a 3D space)

##### The General Rule

- **The linear combination of N x M-dimensional vectors can fill an N-dimensional plane in M-dimensional space**
- When dealing with ***N***-D vectors/spaces, we need at least ***N*** vectors to fill the space through their linear combination.
- If any of the vectors share the same direction, then the dimensionality of the object representing the region covered by possible linear combinations will reduce by 1D.

> Line, Plane, Space

#### Special Linear Combinations

1. Sum: $$1\mathbf{p} + 1\mathbf{q}$$
1. Difference: $$1\mathbf{p} - 1\mathbf{q}$$
1. Zero: $$0\mathbf{p} + 0\mathbf{q}$$ (**zero vector**)
1. Scalar multiple: $$a\mathbf{p} + 0\mathbf{q}$$ (vector $$a\mathbf{p}$$ in the direction of $$\mathbf{p}$$)

> The zero vector is always included in the space of possible outputs

### Dot product == Inner Product == Element-wise product

The dot product of $$\mathbf{p}$$ and $$\mathbf{q}$$, denoted $$c$$ (a scalar) is given by summing the products of the elements of the two vectors.

$$
c = \mathbf{p}\cdot\mathbf{q} = \sum_{d=1}^{D}\mathbf{p}_{d}\mathbf{q}_{d}
$$

- The **dot product** of 2 vectors will always yield a **scalar** value.
- If the dot product between two vectors is equal to 0, then they are ***orthogonal***
- The order of the two vectors does not matter: $$\mathbf{p}\cdot\mathbf{q} = \mathbf{q}\cdot\mathbf{p}$$

### Vector Norm == Vector Length == Vector Magnitude

- The **Norm** or **Magnitude** of a vector is always a scalar value
- Given by the square root of the sum of squared elements (expression below)
- This is equivalent to the root of the dot product of a vector with itself

$$
norm[
\mathbf{p}
]
=
\left\Vert
p
\right\Vert
=
\sqrt{
\mathbf{p} \cdot \mathbf{p}
}
=
\sqrt{
\sum_{d=1}^{D}\mathbf{p}_{d}^2
}
$$

```python
import numpy.linalg as LA
LA.norm(p)

>> 3.7416
```

### ($$L_{2}$$) Distance between two points

- Given by the square root of the sum of squared elements:

$$
d = \sqrt{
\sum_{n=1}^{N}(p_{n} - q_{n})^2
}
$$

- Can also be given by taking the norm of the difference between the vectors:

$$
d =
\left\Vert
p - q
\right\Vert
$$

```python
d1 = np.sqrt(np.sum((p-q)**2))
d2 = LA.norm(p-q)

assert d1 == d2
```

### Unit Vector

- A unit vector has **magnitude** = 1
- Therefore, if $$\mathbf{p}$$ is a unit vector: $$\mathbf{p} \cdot \mathbf{p} = 1$$

### Vector Normalisation

- Normalisation scales a vector so that it has unit length
- To do so, we divide the vector by its length/magnitude: $$\frac{ \mathbf{v} }{ \left\Vert \mathbf{v} \right\Vert }$$

### Cross Product == Vector Product

The cross-product is specialised to 3 dimensions. It can be written:

$$
\mathbf{c} = \mathbf{a} \times \mathbf{b} = \mathbf{A}_{x}\mathbf{b}
$$

which is equivalent to:

$$
\begin{bmatrix}
    c_{1} \\
    c_{2} \\
    c_{3}
\end{bmatrix}
=
\begin{bmatrix}
    0       & -a_{3}    & a_{2} \\
    a_{3}   & 0         & -a{1} \\
    -a_{2}  & a_{1}     & 0
\end{bmatrix}
\begin{bmatrix}
    b_{1} \\
    b_{2} \\
    b_{3}
\end{bmatrix}
$$

And it has the property that the cross product is **orthogonal** to *both* $$\mathbf{a}$$ and $$\mathbf{b}$$

$$
\mathbf{a}^{T}(\mathbf{a} \times \mathbf{b}) = \mathbf{b}^{T}(\mathbf{a} \times \mathbf{b}) = 0
$$

## Matrices & Matrix Operations

### Matrix multiplication


### Matrix Inverse


### Matrix Determinant


### Matrix Trace


### Orthogonal & Rotation Matrices


### Positive Definite Matrices


### Null space of a Matrix

The **right null** space of a matrix $$\mathbf{A}$$ consists of the set of vectors $$x$$ for which $$ \mathbf{Ax} = \mathbf{0} $$

The **left null** space of a matrix $$\mathbf{A}$$ consists of the set of vectors $$x$$ for which $$  x^{T}\mathbf{A} = \mathbf{0}^{T} $$

***What is 0 in this context if we can take it's transpose?***

A square matrix only has a non-trivial null space (ie. not just $$  x = \mathbf{0} $$) if the matrix is singular (non-invertible) & the determinant is therefore 0
