---
layout: post
title: "Retrieving transformation info from instances"
description: ""
category: 
tags: [numerical analysis, matrix calculus, least squares, transformation matrix, instance]
---
{% include JB/setup %}

Suppose you have a huge number of similar objects in world space. These objects can be represented as instances.
But, the meshes of these objects are in world space and are represented as un-related objects.
The meshes are given to be of identical geometry and topology and material assignment.
Also, it is given that, the unknown transformation that happened in creating each of the mesh is of affine nature (
ie: one involving, scale, rotation, shearing and translation ).
Given two similar meshes in world space, how can one guess the transformation that transforms one mesh to another?



Let the vertices of mesh $$ M_0 $$ be $$ a_0, a_1, a_2 ... a_{n-1} $$
and the vertices of mesh $$ M_1 $$ be $$ b_0, b_1, b_2 ... b_{n-1}. $$
Each $$ a_i $$ or $$ b_i $$ is a $$ 1 \times 4 $$ row vector of the $$ x, y , z $$ and $$ w $$ components of the corresponding vertex.
The $$ w $$ component is always $$ 1 $$.

We have
$$ b_i = a_i \times T, i=0 $$ to $$ n-1, $$
where $$ T $$  is an unknown $$ 4 \times 4 $$ matrix.
Please consider $$ T $$ as made up of four $$ 4 \times 1 $$ matrices, $$ T_x, T_y, T_z $$ and $$ T_w $$.

Now

$$ \begin{equation*} b_i.x = a_i \times T_x \label{eq:sample} \end{equation*} $$

$$ \begin{equation*} b_i.y = a_i \times T_y  \end{equation*} $$

$$ \begin{equation*} b_i.z = a_i \times T_z  \end{equation*} $$

$$ \begin{equation*} b_i.w = a_i \times T_w  \end{equation*} $$

Make a big $$ n \times 1 $$ matrix $$ B_x $$, where the $$ i^{th} $$ row of $$ B $$ is $$ b_i.x $$.
Make a big $$ n \times 4 $$ matrix $$ A $$, where the $$ i^{th} $$ row of $$ A $$ is $$ a_i $$.

$$ \begin{equation*} B_x = A \times T_x  \end{equation*} $$

$$ \begin{equation*} B_y = A \times T_y  \end{equation*} $$

$$ \begin{equation*} B_z = A \times T_z  \end{equation*} $$

$$ \begin{equation*} B_w = A \times T_w  \end{equation*} $$

$$ B_x, B_y, B_z , B_w $$ and $$ A $$ are known. $$ T_x, T_y, T_z, Tw $$ are unknown.

Since $$ n $$, the number of vertices is often a large number compared to the other matrix dimension which is $$ 4 $$,
we have an over constrained system (ie too many knowns and too few unknowns).
We can find the value of $$ T $$, which gives the minimum least squares error.

Let $$ e_i $$ be the $$ 1 \times 4 $$ matrix which denotes the error,

$$ \begin{equation}  e_i = [ (b_i.x - a_i \times T_x), (b_i.y - a_i \times T_y), (b_i.z - a_i.z \times T_z), (b_i.w - a_i.w \times T_w) ] \end{equation} $$

Let $$ E_x $$ be the $$ n \times 1 $$ matrix where the $$ i^{th} $$ row is $$ e_i.x $$.

Note that

$$  \begin{equation} e_i.x =  (b_i.x - a_i \times T_x) \end{equation} $$

From $$ (2) $$,

$$  \begin{equation} E_x = B_x - A \times T_x \end{equation} $$

Let "$$ ^T $$" be the transpose operator.

$$  \begin{equation} E_x^T \times E_x = \sum_{i=0}^{n-1} (e_i.x)^2 \end{equation} $$

Similarly

$$ E_y^T \times E_y = \sum_{i=0}^{n-1} (e_i.y)^2 $$

$$ E_z^T \times E_z = \sum_{i=0}^{n-1} (e_i.z)^2 $$

$$ E_w^T \times E_w = \sum_{i=0}^{n-1} (e_i.w)^2 $$

From $$ (3) $$,

$$ \begin{equation*} E_x^T \times E_x = (B_x - A \times T_x)^T \times (B_x - A \times T_x) \end{equation*} $$

$$ \begin{equation} E_x^T \times E_x = (B_x^T \times B_x) - (T_x^T \times A^T \times B_x) - (B_x^T \times A \times T_x) + (T_x^T \times A^T \times A \times T_x) \end{equation} $$

Now $$ T_x^T \times A^T \times B_x $$ and  $$ B_x^T \times A \times T_x $$ are each $$ 1 \times 1 $$ matrices and are the same. So $$ (5) $$ can be written as,

$$ \begin{equation} E_x^T \times E_x = (T_x^T \times A^T \times A \times T_x) - (2 \times T_x^T \times A^T \times B_x) + (B_x^T \times B_x) \end{equation} $$

Differentiate this matrix equation wrt. the unknown, ie $$ T_x $$.
The minimum error occurs when the differentiated result is equal to zero,
and the second differential is a positive definite matrix.

Please use the following equations from matrix calculus is doing the differentiation.
Let $$M$$ be a $$ n \times n $$ matrix and let $$ p $$ be a $$ n \times 1 $$ column vector.

$$ \frac{d(M \times p)}{dp} = M $$

$$ \frac{d(p^T \times M \times p)}{dp} = p^T \times ( M^T + M) $$


$$ \frac{d(p^T \times M )}{dp} = \frac{d(M^T \times p)}{dp} = M^T $$

$$ \frac{d(E_x^T \times E_x)}{dT_x} = 2 \times T_x^T \times ( A^T \times A ) - 2 \times B_x^T \times A $$


Equating this above derivative to zero, we get a linear equation for $$ T_x $$, which we can solve using any of the popular numerical methods for solving linear equations.

$$ \begin{equation} T_x^T \times ( A^T \times A ) =  B_x^T \times A \end{equation}  $$

or

$$ (A^T \times A) \times T_x = A^T \times B_x $$

or

$$ \begin{equation}  T_x = (A^T \times A)^{-1} \times A^T \times B_x  \end{equation} $$

Note that the second differential of $$ (6) $$ is $$ A^T \times A $$ which is a positive definite matrix.
so solving for the above value of $$ T_x $$ will give us the least square minimum.

Similarly we can solve for $$ T_y $$, $$ T_z $$ and $$ T_w $$ separately,
and thusly retrieve $$ T $$.

Ref: Practical Least Squares for Computer Graphics, Siggraph-07 Course.

