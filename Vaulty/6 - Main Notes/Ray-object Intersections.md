2026-06-12 08:31
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Ray-object Intersections

A renderer needs to figure out if, when and were something is hit from the camera. We are interested in what the ray hits first. This is to get the point at which we need to simulate light.

We can use the parametric form of a ray for this. If **r** is a ray and **d** is the direction vector and **o** is the origin at which the ray starts then we have this

$$r(t) = o + t\vec{d}$$
If we use the **t** parameter which goes from $[0, \infty)$ we can travel down an ray to find the point given value **t**.

We can easily find the interaction with a function $F(x, y, z) = 0$ So we substitute the ray equation into the implicit equation and make a function whose only paramter is **t**. We solve this function for **t** and substitute the smallest possible root into the ray equation to find the point.

So in this case our $F(x, y, z) = 0$  function defines a sphere at the origin like this. 
$$x^2 + y^2 + z^2 - r^2 = 0$$
We can throw in our ray equation like this

$$(o_x + t\vec{d}_x)^2 + (o_y + t\vec{d}_y)^2 + (o_z + t\vec{d}_z)^2 - r^2 $$
This give us a quadratic equation in **t**. If there are no real roots, the ray missed. If there are real roots, then we just take the smallest positive integer of **t** to get the interaction point.

So imagine the equation above, now we cast a ray with the equation $$r(t) = o + t\vec{d}$$
Over the quadratic 
$$(o_x + t\vec{d}_x) + (o_y + t\vec{d}_y) + (o_z + t\vec{d}_z) - r^2 $$

and we do this until we have a real root, while increasing **t** by one over time. Once we have a real root, we have an intersection.
# References
##### Main Notes
[[Properties for ray tracers]]
[[How the camera works with ray tracing]]
[[Basics of ray tracing]]
[[How the camera works with ray tracing]]
#### Source Notes
[[Physically Based Rendering - Introduction]]