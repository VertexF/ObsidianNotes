2026-06-12 08:59
Status: #baby 
Tags: [[graphics theory]] [[ray tracing]]
# Light scattering at a surface

With a [[Ray-object Intersections]] found and the [[Light distribution]] worked out and the [[Light visibitlity]] of a surface worked out for all light. We can work out the location and incident lighthing.

We need to work out how much light is scatter back down the ray to the camera. This is the incident lighting effect. 

The incident of light arriving along the casted ray looks like this

![[pha01f09.svg|510]]

Were $\vec{\omega_i}$ interacts with the surface of **P** and it scattered back towards the camera direction $\vec{\omega_o}$. The amount of light scattered toward the camera is given by the product of incident light energy and BRDF.
# References
##### Main Notes
[[Ray-object Intersections]]
[[Light distribution]]
[[Light visibitlity]]
#### Source Notes
[[Physically Based Rendering - Introduction]]