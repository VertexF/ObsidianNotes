2026-06-12 08:37
Status: #baby 
Tags: [[graphics theory]] [[ray tracing]]
# Properties for ray tracers

Just doing [[Ray-object Intersections]] for a given ray is not enough. We need to pass the properies of a given surface to the next part of the ray tracing algorithm. To shade the point we need to pass the geometric information as well. 

We **ALWAYS** need to to pass the surface normal $\hat{n}$ to later parts of the algorithm. We need to worry about partial derivatives of position and surface normal with respect to local parameters of the surface if we can, to be more accurate. However can just pass $\hat{n}$ for a simpler ray tracer.
# References
##### Main Notes
[[The general approach to ray tracing]]
[[Ray-object Intersections]]
#### Source Notes
[[Physically Based Rendering - Introduction]]
