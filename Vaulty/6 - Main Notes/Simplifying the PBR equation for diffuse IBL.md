2026-06-08 10:24
Status: #baby 
Tags: [[graphics theory]] [[physically based renderer]] [[image based lighting]]
# Simplifying the PBR equation for diffuse IBL

If you already have a processed cubemap this gets easier to work out. We can visualise every texel of cubemap as one single emitting light source. Based on the PBR equation we can sample over a cubemap and get our the scene radiance based from that direction.

You want things to be fast but we have a problem you'll need to handle all direction of $\omega_i$ over the entire hemisphere $\Omega$ which is far too expensive to do for each fragment shader invocation. To help with speed you want to pre-compute most of the computation. You can also split the diffuse $k_d$ and specular $k_s$ in terms of BRDF can be made independent from each other.

$$L_o(p,\omega_o) = 
		\int\limits_{\Omega} (k_d\frac{c}{\pi}) L_i(p,\omega_i) n \cdot \omega_i  d\omega_i
		+ 
		\int\limits_{\Omega} (k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
			L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$
If you split of the diffuse and specular parts of the integral you can focus on the diffuse part of the BRDF.

If you look at the lambert term which is a constant term (the colour $c$ the refaction ration $k_d$ and $\pi$) these are not dependent on the integral variables. So you can pull that out like this.

$$L_o(p,\omega_o) = 
		k_d\frac{c}{\pi} \int\limits_{\Omega} L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$
This means that the integral it's only depenent on $\omega_i$ (assuming that $p$ is at the centre of the environment map). With this knowledge, we can calculate or pre-compute a new cubemap that stores in each sample direction (or texel) with $\omega_0$  and the result of the diffuse integral's will result in convolution. 

Convolution in maths measn we are taking 2 function that produces a third as a integral product of two functions $f*g$ if both f and g are functions.

Here what we are doing is considering all the data in a solution set and applying some kind of computation to every piece data in that same set. In our case the set is the scene's radiance or environment map. Meaning when we are considering a single sample from the cubemap we have to consider all other samples that get sampled over the hemasphere.
# References
##### Main Notes
[[The goal with using IBL and PBR]]
[[Making the irradiance map]]
#### Source Notes
[[Diffuse irradiance]]