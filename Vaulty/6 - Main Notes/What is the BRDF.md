2026-06-12 09:02
Status: #baby 
Tags: [[graphics theory]] [[BRDF]]
# What is the BRDF

Each object in a scene has a material tied to it, which describes the apperabce properties at each point on a surface. The **bidirectional reflectance distribution function (BRDF)** tell us how much energy is reflected from the incoming $\vec{\omega_i}$ to an outgoing direction $\hat{\omega_o}$ for [[Light scattering at a surface]]

I will refer to the **BRDF** function as $f_r(P,\vec{\omega_o}, \vec{\omega_i})$ by conversion the direction are unit vector. This really helps with the calculations.

You can easily generalise the notion of BRDF to transmit light or scatter of light from different part of the surface. You also have the more complex **bidirectional scattering distribtion function (BSDF)** which support the scattering of light from a surface. 

A even more complex function is the **bidirectional scattering surface reflectance distribution function (BSSRDF)**, this model and emit light back out that go into the surface and back out. This is necessary to correctly work with translucent materials such as milk, marble or skin. You can fake this with texture it wont respond correctly with light.
# References
##### Main Notes
[[Light scattering at a surface]]
[[Introduction to BRDF]]
#### Source Notes
[[Physically Based Rendering - Introduction]]