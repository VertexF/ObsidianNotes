2026-06-16 14:36
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# Introduction to BRDF

So the **BSDF (Bidirectional Scattering Distribution Function)** is made up for 2 models. The 
**BRDF (Bidirectional Scattering Distribution Function), BRDF (Bidirectional Reflectance Distribution Function)** and the **BTDF (Bidirectional Transmittance Function)**.

So BRDF is made up of two component $f_d$ and $f_r$  this would look something like this with BRDF, note the surface normal $n$ and the $l$

![[diagram_fr_fd.png]]

We would write the BRDF as $f(v, l) = f_d(v, l) + f_r(v, l)$ we of course would need to intergate over a hemisphere to solve this under rendering equation.

A microfacet BRDF this a good BRDF for our purposes. This talks about things aren't smooth at the micro level. They are made up of randonly aligned planar surface fragments called microfacts. Only the microfacet whose normal is oriented halfway between the light direction and view direction will reflect visible light. ![[diagram_macrosurface.png]]

Not all microfacets with a properly oriented normal will contribute reflect because of shadow and masking.![[diagram_shadowing_masking.png]]

With increased roughness the specular highlight blur as the light is more reflected scattered away from the camera. ![[diagram_roughness.png]]

So here the render equation again but with a BRDF and the $\cos()$ as a dot product instead.
$$f_x(v,l) = \frac{1}{|n\cdot v||n\cdot l|} \int_\Omega D(m,\alpha)G(v,m,l)f_m(v,l,m)(v\cdot m)(l \cdot m)dm$$
The term $D$ models the distribution of the microfacets. This term is also called the **Normal Distribution Function (NDF)** The term play a primordial role in the apperance of surfaces, as you can see in the image above.

The term $G$ models the visibility is the occluded or shadow masked part of the microfacets.

To simplify things we make that when a light ray hit a surface we assume that fragment were it lands is also flat, meaning that a single light ray fires from that surface. 
# References
##### Main Notes
[[What is the BRDF]]
#### Source Notes
[[Filament PBR - Materials]]