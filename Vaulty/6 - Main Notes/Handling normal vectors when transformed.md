2026-07-02 11:14
Status: #baby 
Tags: [[maths]]
# Handling normal vectors when transformed

We are going to make a sample problem below to explain the issues with normal vectors and transforming them into different space.

If we have a vector $\vec{v^A}$ (were $A$ is a space) if we were transform  $\vec{v^A}$ my matrix $M$ then the $x^A$ of vector $\vec{v^A}$ add up to total length of $x^A = \vec{x^B} + \vec{y^B} + \vec{z^B}$. We can express $\vec{v^B}$ using $\vec{v^A}$ in the $B$ cooridnate system.
$$\left(\frac{\Delta \vec{x}_{}^{B}}{\Delta \vec{x}_{}^{A}}\vec{v}_{x}^{A}, \frac{\Delta \vec{y}_{}^{B}}{\Delta \vec{x}_{}^{A}}\vec{v}_{x}^{A}, \frac{\Delta \vec{z}_{}^{B}}{\Delta \vec{x}_{}^{A}}\vec{v}_{x}^{A} \right)$$
We can do the same for $y$ and $z$ as we did for $x$ here. We can building a tranformed vector $\vec{v}_{}^{B}$ that corrisponds to the orignal vector $\vec{v}_{}^{A}$ in it's entirety, this can be written as a matrix tranformation

![[matrix-mult-to-B-2.png]]

We are just creating a matrix that transform $\vec{v}_{}^A$ to $\vec{v}_{}^B$ based on the definitation we defined above.

If we consider a normal vector we calculate things as a gradient. The key understnding is how such normal vectors transform is realising that components of a gradient do not measure distance, but the **reciprocal** distance. 

This is because you consider the partial derivatives that compose a normal vector, which consider a point on a surface $(\delta f / \delta x, \delta f / \delta y, \delta f / \delta z)$ the distance $x, y, z$ are all in the denomiator of the vector. Since this is so different we write the normal vector a row matrix rather than a column matrix for a regular vector.
$$\vec{n} = \left[ \frac{1}{n_{x}} \frac{1}{n_{y}} \frac{1}{n_{z}}\right]$$
We need a matrix that can when inverted can reverse the tranformation normal vector from 1 transform back to another. We also need be able to do a $\vec{n} \vec{t} = 0$ which is a column x row matrix multiplication, which is often incorrected referred to as a dot product.

So this comes down to this. If we have a matrix that transforms vectors to coordinate system A to B. Then $n_{}^{B}$ is given by $n^A M^{-1}$ and transformed tragent $t^B$ is given by $Mt^A$ give the product.
$$n^B \cdot t^B = n^t M^{-1}Mt^A = n^A \cdot t^A = 0$$
and this means that $\vec{t}$ and $\vec{n}$ are still perpendicular in coordinate system B after the transform of $M$. This is the problem you have to solve to transform normal vector correctly. Note this is totally correct BUT not all matrices can be inverted.
# References
##### Main Notes
[[The adjungate normal matrix]]
[[Normal vector maths]]
[[The problem with transform normal vectors]]
#### Source Notes
[[Foundations of Game Engine Developemt - Maths]]