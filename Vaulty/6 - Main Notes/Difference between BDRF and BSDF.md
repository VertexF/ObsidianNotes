2026-06-20 08:49
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# Difference between BDRF and BSDF

That the [[What is the light transport equation]] is for a full sphere because it's doing sub-surface scattering rather than the BRDF which in rasterisation graphics, you are all only interested in how much light is reflected back at the camera. Meaning that you do the internal over a hemisphere not a full sphere.

This means that the light transport equation here. 
$$L_o(P, \vec{\omega_o}) = L_e(P, \vec{\omega_o}) + \int\limits_{S^2} f(P,\vec{\omega_o}, \vec{\omega_i}) L_i(P,\vec{\omega_i}) | \cos(\theta_i) | d\vec{\omega_i}$$
Is more like this
$$L_o(p,\omega_o) = \int\limits_{\Omega} 
    	(k_d\frac{c}{\pi} + k_s\frac{DFG}{4(\omega_o \cdot n)(\omega_i \cdot n)})
    	L_i(p,\omega_i) n \cdot \omega_i  d\omega_i$$
Where the $n \cdot \omega_i$ is the $cos(\theta_{i})$ but because it's over a hemisphere it should be clamped to 0 and 1 because we aren't interested in anything on the otherside of the sphere. 

# References
##### Main Notes
[[Direct lighting theory]]
#### Source Notes
