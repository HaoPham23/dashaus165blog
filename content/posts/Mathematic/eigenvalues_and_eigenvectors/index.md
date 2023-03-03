---
title: Eigenvalues and Eigenvectors
subtitle:
date: 2023-03-02T23:23:22+07:00
draft: false
description:
keywords:
license:
comment: false
weight: 0
tags:
  - Math
categories:
  - Mathematic
hiddenFromHomePage: false
hiddenFromSearch: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: true
lightgallery: false
password:
message:
repost:
  enable: false
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content/#front-matter
---

{{< admonition >}}
This article is written by chatGPT. This is for catergories testing purpose.
{{< /admonition >}}


# Introduction to Eigenvalues and Eigenvectors

Eigenvalues and eigenvectors are important concepts in linear algebra that are used to study transformations and matrices. They are particularly useful in applications such as physics, engineering, and computer graphics.

Definition of Eigenvalues and Eigenvectors
Let A be an n x n matrix. An eigenvector of A is a non-zero vector x that satisfies the following equation:

$$A\mathbf{x} = \lambda \mathbf{x}$$

where $\lambda$ is a scalar called the eigenvalue corresponding to x.

In other words, when A is multiplied by an eigenvector x, the result is a scaled version of x. The scaling factor is the corresponding eigenvalue $\lambda$.

## Examples of Eigenvalues and Eigenvectors

Consider the following 2 x 2 matrix A:

$$A = \begin{pmatrix} 2 & 1 \\ 1 & 2 \end{pmatrix}$$

To find the eigenvalues and eigenvectors of A, we solve the equation:

$$(A - \lambda I)\mathbf{x} = \mathbf{0}$$

where I is the identity matrix and $\mathbf{0}$ is the zero vector. This leads to the following characteristic equation:

$$(2 - \lambda)(2 - \lambda) - 1 = 0$$

Solving for $\lambda$, we get $\lambda_1 = 3$ and $\lambda_2 = 1$. To find the corresponding eigenvectors, we substitute each eigenvalue into the original equation:

$$(A - 3I)\mathbf{x}_1 = \mathbf{0}$$

$$(A - I)\mathbf{x}_2 = \mathbf{0}$$

Solving these equations, we get:

$$\mathbf{x}_1 = \begin{pmatrix} 1 \ -1 \end{pmatrix} \quad \text{and} \quad \mathbf{x}_2 = \begin{pmatrix} 1 \ 1 \end{pmatrix}$$

These eigenvectors are scaled by their corresponding eigenvalues:

$$A\mathbf{x}_1 = 3\mathbf{x}_1 \quad \text{and} \quad A\mathbf{x}_2 = \mathbf{x}_2$$

$$
\begin{bmatrix}
    a_{11} & a_{12} & \dots  & a_{1n} \\
    a_{21} & a_{22} & \dots  & a_{2n} \\
    \vdots & \vdots & \ddots & \vdots \\
    a_{m1} & a_{m2} & \dots  & a_{mn}
\end{bmatrix}
\begin{bmatrix}
    x_{1} \\
    x_{2} \\
    \vdots \\
    x_{n}
\end{bmatrix}
=
\begin{bmatrix}
    b_{1} \\
    b_{2} \\
    \vdots \\
    b_{m}
\end{bmatrix}
$$

## Properties of Eigenvalues and Eigenvectors

There are several important properties of eigenvalues and eigenvectors that are useful in applications:

The sum of the eigenvalues of a matrix A is equal to the trace of A (the sum of the diagonal elements).
The product of the eigenvalues of A is equal to the determinant of A.
If A is symmetric, then its eigenvectors are orthogonal to each other.
If A is invertible, then its eigenvectors form a basis for the space.

## Conclusion

Eigenvalues and eigenvectors are important tools in linear algebra that allow us to study transformations and matrices in a more systematic way. They have many practical applications in fields such as physics, engineering, and computer graphics. By understanding the properties of eigenvalues and eigenvectors, we can gain insight into the behavior of complex systems and make predictions based on data.

<!--more-->
