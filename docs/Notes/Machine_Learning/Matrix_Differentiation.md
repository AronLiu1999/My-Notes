---
comments: true
---

# Matix Differntiation

## Scalar Function

### Vector Derivative of a Scalar Function

Let $y = f(x_1, x_2, ..., x_n)$, we have the total differential of $y$ as

$$
d y = \frac{\partial y}{\partial x_1} d x_1 + \frac{\partial y}{\partial x_2} d x_2 + ... + \frac{\partial y}{\partial x_n} d x_n
$$

Let $d \mathbf{x} = [dx_1, dx_2,...,dx_n]^T$ and $\nabla_{\mathbf{x}} f=\left[\frac{\partial f}{\partial x_1} \frac{\partial f}{\partial x_2} \frac{\partial f}{\partial x_3}\right]^T$, so we have

$$
d y = \langle \nabla_{\mathbf{x}} f, d \mathbf{x} \rangle = \nabla_{\mathbf{x}} f^T d \mathbf{x}
$$

???+ note "Differential V.S. Derivative"
    - Differential: the infinitesimal difference in varying variables
    - Derivative: the rate of change of a function with respect to the changes variables

### Matrix Derivative of a Scalar Function

Let $y = f(\mathbf{X})$, where $\mathbf{X}$ is a $m \times n$ matrix, we have the total differential of $y$ as

$$
d f=\sum_{i=1}^m \sum_{j=1}^n \frac{\partial f}{\partial X_{i j}} d X_{i j}
$$

Let $d \mathbf{X}$ be the matrix of differential $d X_{i j}$ for all $i, j$, i.e., $d \mathbf{X} = [d X_{i j}]_{m \times n}$

Let $\nabla_\mathbf{{X}} f$ be the derivate of $f$ w.r.t $\mathbf{X}$, i.e., $\nabla_\mathbf{{X}} f = [\frac{\partial f}{\partial X_{i j}}]_{m \times n}$

Then we have

$$
d y = \langle \nabla_{\mathbf{x}} f, d \mathbf{X} \rangle = \operatorname{Tr} \left(\nabla_\mathbf{X} f ^ T d \mathbf{X}\right)
$$

the $\operatorname{Tr} \left(\cdot \right)$ is defined ans

$$
\operatorname{Tr} \left(\mathbf{A}^T \mathbf{B} \right) = \sum_{i j} A_{i j} B_{i j}
$$

hence we have

$$
d y = \operatorname{Tr} \left(\nabla_\mathbf{X} f ^ T d \mathbf{X}\right)=\operatorname{Tr}\left(\frac{\partial f}{\partial \mathbf{X}}^T d \mathbf{X}\right)
$$

### Basic Rules for Matrix Differentiation

1. $d \left( \mathbf{X} + \mathbf{Y} \right)  = d \mathbf{X} + d \mathbf{Y}$
2. $d \left(\mathbf{X Y} \right) = \left( d\mathbf{X}\right) \mathbf{Y} + \mathbf{X}d\mathbf{Y}$
3. $d\left(\mathbf{X}^T\right) = \left(d\mathbf{X} \right)^ T$
4. $d\operatorname{Tr}\left(\mathbf{X}\right)  = \operatorname{Tr}\left(d\mathbf{X} \right)$
5. $d \mathbf{X}^{-1} = -\mathbf{X}^{-1} \left(d \mathbf{X}\right)\mathbf{X}^{-1}$
6. $d|X|=\operatorname{Tr}\left(\mathbf{X}^{\#} d \mathbf{X}\right)$
7. Hadamard Product: $d \left( \mathbf{X} \circ \mathbf{Y}\right) = d \mathbf{X} \circ \mathbf{Y} + \mathbf{X} \circ d \mathbf{Y}$
8. Kronecker product: $d \left( \mathbf{X} \otimes \mathbf{Y}\right) = d \mathbf{X} \otimes \mathbf{Y} + \mathbf{X} \otimes d \mathbf{Y}$
9. $d \sigma \left( \mathbf{X} \right) = \sigma '\left(\mathbf{X} \right) \otimes d \mathbf{X}$, where $\sigma(\mathbf{X}) = [\sigma(\mathbf{X}_{i j})]$

???+ question "How to prove $d \mathbf{X}^{-1} = -\mathbf{X}^{-1} \left(d \mathbf{X}\right)\mathbf{X}^{-1}$"

    $$
    \mathbf{X} \mathbf{X}^{-1} = \mathbf{I}
    $$

    $$
    d \mathbf{X} \mathbf{X}^{-1}  = \left(d \mathbf{X} \right) \mathbf{X} ^ {-1} + \mathbf{X} d \mathbf{X} ^ {-1} = d \mathbf{I} = 0
    $$

## Matrix Function

### Scalar Derivative of a Matrix Function

Let $\mathbf{y} = f(x)$, where $\mathbf{y}$ is a vector, the derivative of $\mathbf{y}$ w.r.t scalar $x$ is

$$
\frac{d \mathbf{y}}{d x} = \frac{d f(x)}{dx} = f' (x)
$$

where $f'(x)$ is a vector

The differential of $\mathbf{y}$ ,i.e. $d \mathbf{y}$ is a vector

$$
d \mathbf{y} = f'(x) d x
$$

### Vector Derivative of a Vector Function

Let $\mathbf{y} = f \left( \mathbf{x} \right)$, where $\mathbf{y}$ is $m \times 1$ vector and $\mathbf{x}$ is $n \times 1$ vector, we have the total differential of $\mathbf{y}$ as

$$
\nabla_{\mathbf{x}}f = \frac{\partial f}{\partial \mathbf{x}}=\left[\begin{array}{cccc}
\frac{\partial y_1}{\partial x_1} & \frac{\partial y_2}{\partial x_1} & \cdots & \frac{\partial y_m}{\partial x_1} \\
\frac{\partial y_1}{\partial x_2} & \frac{\partial y_2}{\partial x_2} & \cdots & \frac{\partial y_m}{\partial x_2} \\
\vdots & \vdots & \ddots & \vdots \\
\frac{\partial y_1}{\partial x_n} & \frac{\partial y_2}{\partial x_n} & \cdots & \frac{\partial y_m}{\partial x_n}
\end{array}\right]_{n \times m}
$$

hence, we have the total differential as

$$
d \mathbf{y} = \nabla_{\mathbf{x}}f ^ T d \mathbf{x} = \frac{\partial {\boldsymbol{f}}^T}{\partial \mathbf{x}} d \mathbf{x}
$$

### Matrix Derivative of a Matrix Function

Let $\mathbf{Y} = f(\mathbf{X})$, where $\mathbf{X}$ is a $m \times n$ matrix and $\mathbf{Y}$ is a $p \times q$ matrix. The gradient of $\mathbf{Y}$ w.r.t $\mathbf{X}$ can be defined using the definination of the differential w.r.t a vector.

So, we first vectorization the matrix, then the gradient of $\mathbf{Y}$ w.r.t $\mathbf{X}$ is

$$
 \nabla_{\mathbf{X}} \mathbf{Y} = \frac{\partial \mathbf{Y}}{\partial \mathbf{X}} = \nabla_{\operatorname{vec}(\mathbf{X})} \operatorname{vec}(\mathbf{Y}) = \frac{\partial \operatorname{vec}(\mathbf{Y})}{\partial \operatorname{vec}(\mathbf{X})}
$$

???+ note "The Definition of Vectorization"
    Given $\mathbf{X} \in \mathbb{R}^{m \times n}$, $\operatorname{vec}(\mathbf{X})$ is an $mn \times 1$ vector, where $\operatorname{vec}(\mathbf{X})$ is defined as

    $$ \operatorname{vec} (\mathbf{X}) = [X_{11}, X_{21}, \cdots ,X_{m1}, X_{21},X_{22},  \cdots, X_{m2}, \cdots, X_{1n}, X_{2n}, \cdots , X_{mn}]
    $$

The total differential of $\operatorname{vec}(\mathbf{Y})$ w.r.t $\operatorname{vec}(\mathbf{X})$ is

$$
\operatorname{vec}( d \mathbf{Y}) = (\nabla_{\mathbf{X}} \mathbf{Y})^T \operatorname{vec} (d \mathbf{X}) = (\frac{\partial \mathbf{Y}}{\partial \mathbf{X}})^T \operatorname{vec} (d \mathbf{X})
$$

### Some Rules derived from vectorization

- $\operatorname{vec}(\mathbf{AXB}) = (\mathbf{B}^T \otimes \mathbf{A}) \operatorname{vec}(\mathbf{X})$
- $\operatorname{vec}(\mathbf{X}^T)  = \mathbf{C}_{mn} \operatorname{vec}(\mathbf{X})$

    where $\otimes$ is Kronecker product and $\mathbf{C}_{mn}$ is commutation matrix, it can be computed by

$$
\mathbf{C}_{mn} = \sum_{i=1}^m \sum_{j=1}^n E_{ij} (m) \otimes E_{ji} (n)
$$

## Unique Skill

https://www.matrixcalculus.org/
