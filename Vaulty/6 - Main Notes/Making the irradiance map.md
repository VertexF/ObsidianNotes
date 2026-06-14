2026-06-08 10:25
Status: #baby 
Tags: [[graphics theory]] [[physically based renderer]] [[image based lighting]]
# Making the irradiance map

We'll want to do the convolution by solving the integral for each output $\omega_0$ sample direction. We do this by grabbing a bunch of $\omega_i$ and work them out and sample the average, over the hemisphere $\Omega$ . The hemisphere we need to build up is a bunch of sample in the direction of $\omega_i$ and it oriented towards the output sample of $\omega_0$ direction we are convoluting.

![[ibl_hemisphere_sample.png|348]]
For each sampled direction $\omega_0$ store the result of the diffuse integral. This is a pre-computed sum of all the indirect diffuse light hitting a surface aligned along the direction $\omega_0$ When you work this out you have a irradiance map. This is because we have a convoluted cubemap which effiectly gives us a sample of the scene's irradiance from any direction $\omega_0$

If you're indoors it's going to be common if you have 1 irradance map at point $p$ you're diffuse light calculation is going to break down. How engines deal with this is to have reflection at different point and they caculated there own irradance map and then you do linear interpretation between each probe.

This is what an irradance map looks like based using this cubemap![[ibl_irradiance.png]]

If we store the convoluted result in each of our cubemap texels (in the direction of $\omega_0$) is kind of like an average of the light and colour. This give us our scene irradiance.
# References
##### Main Notes
[[What is Image Based Lighting]]
[[The goal with using IBL and PBR]]
[[Simplifying the PBR equation for diffuse IBL]]
#### Source Notes
[[Diffuse irradiance]]