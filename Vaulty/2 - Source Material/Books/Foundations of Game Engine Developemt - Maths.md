# Reference Foundations of Game Engine Developemt

##### Triangle Mesh
A closed mesh most for the **Euler formula** which states
$$V - E + F = 2$$
So $V$ is the number of vertices, $E$ is the number of edges and $F$ is the number of faces. In game development when we talk about faces we are only with triangles. Even if triangles are coplanar we still count those edges.
##### Normal vector
So the normal given a scale function $f(p)$ and the normal vector at position $p = (x, y, z)$ is given by the gradient $\Delta f(p)$ because it perpendicular to every tangent direction level to the surface of $f$ at point $p$. For example if we have an ellipsoid
$$f(p) = x^2 + \frac{y^2}{4}+z^2-1 = 0$$

The point $p = (\frac{\sqrt{6}}{4}, 1, \frac{\sqrt{6}}{4})$ lies on the furface of the ellipsoid, and normal vector $\vec{n}$ at the point give.
$$\vec{n} = \Delta f(p) = (\frac{\delta f}{\delta x}, \frac{\delta f}{\delta x}, \frac{\delta f}{\delta x}) |_{p} = (2x, \frac{y}{2}, 2z) = (\frac{\sqrt{6}}{2}, \frac{1}{2}, \frac{\sqrt{6}}{2})$$
Normally we don't do this outside of modeling software which approximate an ideal surfae described by some maths formal.

So if you want to calculate the normal vector we can uses the triangles vertices with a cross project. We just have to create tangent vectors from the points.
$$\vec{n} = (p_{1} - p_{0}) \times(p_{2} - p_{0})$$
If you do the cross product differently you can end up a normal vector that faces inward. If you want to make sure that it's the correct way around the first substraction of points should be choosen in anit-clockwise direction.
##### Transforming normal vectors
Tangent vectors aren't affected by a matrix transform because they are caculated based on the vertices of a model. Meaning that when $p_{1}$ and $p_{2}$ are tranformed by matrix $M$ then the tangent vector $t$ is just $Mt$. However, normal vectors can't be transformed in the same way.

If we scale a model non-uniformaly this scales normal vectors in away that destroy perpendicularity.
##### Handling normal vectors when transformed
If we have a vector $\vec{v}_{A}$ if we were transform  $\vec{v_{A}}$ my matrix $M$ then the $x_{A}$ of vector $\vec{v}_{A}$ add up to total length of $x_{A} = \vec{x}_{B} + \vec{y}_{B} + \vec{z}_{B}$. This is just used to make a mathematical point nothing that's alway. We can express $\vec{v}_{B}$ using $\vec{v}_{A}$ in the $B$ cooridnate system.
$$\left(\frac{\Delta \vec{x}_{}^{B}}{\Delta \vec{x}_{}^{A}}\vec{v}_{x}^{A}, \frac{\Delta \vec{y}_{}^{B}}{\Delta \vec{x}_{}^{A}}\vec{v}_{x}^{A}, \frac{\Delta \vec{z}_{}^{B}}{\Delta \vec{x}_{}^{A}}\vec{v}_{x}^{A} \right)$$
We can do the same for $y$ and $z$ as we did for $x$ here. We can building a tranformed vector $\vec{v}_{}^{B}$ that corrisponds to the orignal vector $\vec{v}_{}^{B}$ in it's entirely, ths can be written as a matrix tranformation

![[matrix-mult-to-B-2.png]]

We are just creating a matrix that transform $\vec{v}_{}^A$ to $\vec{v}_{}^B$ based on the definitation we defined above.

If we consider a normal vector we calculate things as a gradient. The key understnding how such normal vectors transofr is realising that components of a gradient do not measure distance, but the **reciprocal** distance. 

This is because you consider the partial derivatives that compose a normal vector, which consider a point on a surface $(\delta f / \delta x, \delta f / \delta y, \delta f / \delta z)$ the distance $x, y, z$ are all in the denomiator of the vector. Since this is so different we write the normal vector a row matrix rather than a column matrix for a regular vector.
$$\vec{n} = \left[ \frac{1}{n_{x}} \frac{1}{n_{y}} \frac{1}{n_{z}}\right]$$
We need a matrix that can when inverted can reverse the tranformation normal vector from 1 transform back to another. We also need be able to do a $\vec{n} \vec{t} = 0$ which is a column x row matrix multiplication, which is often incorrected referred to as a dot product.

So this comes down to this. If we have a matrix that vectors to coordinate system A to B. Then $n_{}^{B}$ is given by $n^A M^{-1}$ and transformed tragent $t^B$ is given by $Mt^A$ give the product.
$$n^B \cdot t^B = n^t M^{-1}Mt^A = n^A \cdot t^A = 0$$
and this means that they are still $t$ and $n$ are still perpendicular in coordinate system B after the transform of $M$.
##### The adjugate normal matrix
So since we are aware that the normal vector stores the gradients and not the distance to scale, we need to treat the normal vector like a 1 colume row matrix, as described in [[Handling normal vectors when transformed]] To do this we can just invert the matrix.

If we two vector $\vec{t}$ and $\vec{s}$ which are two tangent vector, we can take these and do a cross project and get normal vector. Assume we take into account this $Mv = v_{x}\vec{a} + v_{y}\vec{b} + v_{y}\vec{c}$  were a, b and c are all colume vectors from matrix $M$ we can rewrite out $n^B = Ms \times Mt$ as
$$n^B = (s_{x}M_{[0]} + s_{y}M_{[1]} + s_{z}M_{[2]}) \times (t_{x}M_{[0]} + t_{y}M_{[1]} + t_{z}M_{[2]})$$
$M_{j}$ are column matrices. We can simply things down by distributing the cross product to all these terms we get.
$$n^B = (s_{y}t_{z} - s_{z}t_{t})(M_{[1]} \times M_{[2]})$$
$$= (s_{z}t_{x} - s_{x}t_{z})(M_{[2]} \times M_{[0]})$$
$$= (s_{x}t_{y} - s_{y}t_{x})(M_{[0]} \times M_{[1]})$$
That's for each component of $n$ 

The cross project of $n^A = s \times t$ actually creates the determinate of $M^{-1}$ if we follow the equation to work out the inverse matrix.
$$M^{-1} = \frac{1}{[a, b, c]} [\begin{matrix} b \times c \\ c \times a \\ a \times b \end{matrix}]$$
Which is $det(M)M^{-1}$ We can conclude that a normal vector calculated with a cross product is correctly tranformed according to
$$n^B = n^A det(M)M^{-1}$$
Using the adjungate of M matrix can be current as
$$n^B = n^A adj({M})$$

This is not only how normal vectors transform but how any vector result from a cross product transform betweeen regular vectors. Since we are dealing with normal length the size of the determinate is inconsequential and is often ignored matrix the making $n^B = n^A det(M)M^{-1}$ and $n^B = n^A adj({M})$ are basically the same.

Also not all matrices can be inverted when the determinate of the matrix is 0 because you get a divsion by 0. So this is faster and it's correct too.

This would look like this if we where to write it out in glsl.
```c
mat3 adjungate(in mat4 m)
{
    return mat3(cross(m[1].xyz, m[2].xyz), //b x c
                cross(m[2].xyz, m[0].xyz), //c x a 
                cross(m[0].xyz, m[1].xyz));//a x b
}

void main()
{
	vNormal = mat3(adjungate(modelPostion)) * normal;
}
```

With one expection, when you do a reflecting vertices the winding get reversed because you do a cross product between $t$ and $s$ would be reversed by the determinate $n^B = n^A det(M)M^{-1}$ So be careful with reflecting vertices as it reverse the winding.
##### Parametric lines
Given twos points $p_{1}$ and $p_{2}$ you get this function.
$$L(t) = (1 - t)p_{1} + tp_{2}$$
For rays this would be defined over $[0, \infty)$ and it would be $(-\infty, \infty)$ for a line we can easily re-write into this formula.
$$L(t) = p + t\vec{v}$$ Were $v$ is just $p_{2} - p_{1}$ it's often the case that $\vec{v}$ is a normalised vector so $t$ corresponds to the length of $v$.
##### Vector projection
Sometimes we need to be able to work out the 2 vectors that make up 1 vector by decomposing them into seperate components. You can just get the length of the $x, y$ and $z$ axis to get the individual components of vectors.

It's important to note the basics to projects gets easier for general. If you have basis vectors for a space $\vec{i}, \vec{j}$ and $\vec{k}$ we take the dot project of $(\vec{v} \cdot \vec{i})\vec{i}$ and the same for $j$ and $k$. This is case for when things are normalised. However to project properly we can use just the general equation as
$$a_{||b} = \frac{\vec{a} \cdot \vec{b}}{||b||^2} \vec{b}$$
Here we are project a vector from $\vec{a}$ onto $\vec{b}$ We are dividing by $b^2$ because $\vec{b}$ may not be a normal vector. If the dot product is negative meaning that angle $\theta$ between $\vec{a}$ and $\vec{b}$ is greater than 90 degree then it will project negative.
##### Vector rejection
Vector rejection is just [[Vector projection]] minus the orignal vector you projected from. So took $\vec{a}$ and projected on to $\vec{b}$ we can find the vector rejection by using this formula.
$$\vec{a}_{\bot b} = \vec{a} - \vec{a}_{||b} = \vec{a} - \frac{\vec{a} \cdot \vec{b}}{||b||^2} \vec{b}$$

The notation that $\vec{a}_{\bot b}$ 

This picture explains vector rejection and projection being part of a right angle triangle.
![[vector-projection-rejection-2.png]]
##### Distance between a point and a line
If you want to find the smallest distance $d$ for a given point $A$ using the $L(t) = p + t\vec{v}$ function. To make things easier we define another vector which is calcuated with $\vec{u} = A - p$ were $p$ is the origin point of the line.

So the $d$ is equal to the magnitude of the rejection of $\vec{u}$ onto $\vec{v}$ which corrisponds to the line.
$$d^2 = u^2-(\vec{u}_{||v}) = u^2 - \left(  \frac{\vec{u} \cdot \vec{v}}{||v||^2} \right)^2$$
Simplying the terms corresponds to just taking square root.
$$d = \sqrt{u^2 - \left(  \frac{\vec{u} \cdot \vec{v}}{||v||^2} \right)^2}$$
