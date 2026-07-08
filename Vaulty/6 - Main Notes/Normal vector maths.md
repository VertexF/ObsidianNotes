2026-07-02 11:03
Status: #baby 
Tags: [[maths]]
# Normal vector maths

So the normal vectors given a scale function $f(p)$ and the normal vector at position $p = (x, y, z)$ is given by the gradient $\Delta f(p)$ because it perpendicular to every tangent direction level to the surface of $f$ at point $p$. For example if we have an ellipsoid
$$f(p) = x^2 + \frac{y^2}{4}+z^2-1 = 0$$

The point $p = (\frac{\sqrt{6}}{4}, 1, \frac{\sqrt{6}}{4})$ lies on the surface of the ellipsoid, and normal vector $\vec{n}$ at the point gives.
$$\vec{n} = \Delta f(p) = (\frac{\delta f}{\delta x}, \frac{\delta f}{\delta y}, \frac{\delta f}{\delta z}) |_{p} = (2x, \frac{y}{2}, 2z) = (\frac{\sqrt{6}}{2}, \frac{1}{2}, \frac{\sqrt{6}}{2})$$
Normally we don't do this outside of modeling software which approximate an ideal surfae described by some maths formulas.

So if you want to calculate the normal vector we can uses the triangles vertices with a cross projection. We just have to create tangent vectors from the points.
$$\vec{n} = (p_{1} - p_{0}) \times(p_{2} - p_{0})$$
If you do the cross product differently you can end up a normal vector that faces inward. If you want to make sure that it's the correct way around the first substraction of points should be choosen in anit-clockwise direction.
# References
##### Main Notes
[[Triangle mesh maths]]
#### Source Notes
[[Foundations of Game Engine Developemt - Maths]]