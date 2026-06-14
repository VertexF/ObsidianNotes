2026-06-08 10:21
Status: #baby 
Tags: [[graphics theory]] [[physically based renderer]] [[image based lighting]]
# The goal with using IBL and PBR

We can start to introduce the PBR with IBL if we look at the PBR equation:
$$L_o(p,\omega_o) = \int\limits_{\Omega} 
    	(k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
    	L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$

So the goal here is the do the integral over all lighting direction $\omega_i$ over the hemisphere $\Omega$. Because we need to consider every light direction solving the integral this is non-trivial, unlike when you know all the direction the light is coming in.

We need to do these two task to simplify our calculations.
1) The first task is to grab all the scene radiance given any direction vector $\omega_i$
2) It needs to be fast.
# References
##### Main Notes
[[What is Physically Based Rendering]]
[[Simplifying the PBR equation for diffuse IBL]]
[[What is Image Based Lighting]]
#### Source Notes
[[Diffuse irradiance]]