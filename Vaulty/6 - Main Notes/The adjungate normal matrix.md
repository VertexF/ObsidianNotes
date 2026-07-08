2026-07-02 11:33
Status: #baby 
Tags: [[maths]]
# The adjungate normal matrix

So since we are aware that the normal vector stores the gradients and not the distance to scale, we need to treat the normal vector like a 1 colume row matrix, as described in [[Handling normal vectors when transformed]] To do this we can just invert the matrix.

If we two vector $\vec{t}$ and $\vec{s}$ which are two tangent vector, we can take these and do a cross project and get normal vector. Assumming we take into account this $Mv = v_{x}\vec{a} + v_{y}\vec{b} + v_{y}\vec{c}$  were a, b and c are all colume vectors from matrix $M$ we can rewrite out $n^B = Ms \times Mt$ as
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

This is not only how normal vectors transform but how any vectors result from a cross product transform betweeen regular vectors. Since we are dealing with normal length vectors the size of the determinate is inconsequential and is often ignored matrix the making $n^B = n^A det(M)M^{-1}$ and $n^B = n^A adj({M})$ are basically the same.

Also not all matrices can be inverted when the determinate of the matrix is 0 because you get a divsion by 0. So this is faster and it's correct to use the adjungate.

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

With one expection, when you do a reflect vertices the winding get reversed because the cross product between $\vec{t}$ and $\vec{s}$ the reversal would be correct for by the determinate $n^B = n^A det(M)M^{-1}$ So be careful with reflecting vertices as it reverse the winding.
# References
##### Main Notes
[[Normal vector maths]]
[[The problem with transform normal vectors]]
#### Source Notes
[[Foundations of Game Engine Developemt - Maths]]
