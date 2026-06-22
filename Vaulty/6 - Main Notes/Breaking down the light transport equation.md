2026-06-12 15:08
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Breaking down the light transport equation
 
Here we are breaking the **light transport equation** to make it actually possible to compute. We are going to use random samplying and return values for both of the [[What is the light transport equation|light transport equation]]. Here we are using the BSDF not the BRDF function.

$$L_o(P, \vec{\omega_o}) = L_e(P, \vec{\omega_o}) + \int\limits_{S^2} f(P,\vec{\omega_o}, \vec{\omega_i}) L_i(P,\vec{\omega_i}) | \cos(\theta_i) | d\vec{\omega_i}$$

It's possible with the the first part $L_e(P, \vec{\omega_o})$ **emitted radiance funcion** of the equation for a given ray that the ray itself might not hit anything, however the global light source that lights up everything and has no geometry to be hit, meaning that we need to add light to the scene via this ray based on the "infinite" light sources. 

The second part $\int\limits_{S^2} f(P,\vec{\omega_o}, \vec{\omega_i}) L_i(P,\vec{\omega_i}) | \cos(\theta_i) | d\vec{\omega_i}$ has a lot more going on so now we use a random direction vector $\vec{\omega'}$ . We do this because we have to work out the light value with the BSDF over the hemisphere which is extremely expensive when considering EVERY possible ray. 

So by selecting a random direction of $\vec{\omega'}$ we have equal propability over all possible directions. After that we can estimate the integral and we can compute the weighted product of the BSDF which describe the light scattering properties of the material a point **P**, the $L_i(P,\vec{\omega_i})$ the **incident spectral radiance** and the cosine factor for the angle of the surface at point **P**
$$\int\limits_{S^2} f(P,\vec{\omega_o}, \vec{\omega_i}) L_i(P,\vec{\omega_i}) | \cos(\theta_i) | d\vec{\omega_i} \approx \frac{f(P, \vec{\omega_o}, \vec{\omega'})L_i(P, \vec{\omega'})|\cos(\theta')|}{1/4\pi} $$In other words we randomly get the direction $\vec{\omega'}$ estimating the value of the integral requires evaluating the terms in the intergrand for the direction then scale that by $4\pi$.

The intergrand is just everything after the intergal and before the differential part in an intergral. 

Since you only considering 1 random direction there are error in the Monte Carlo approach, compared to the true intergal. This is the reason why we take multiple samples per pixel because on average we will get a mostly correct result. 

Solving the **BSDF** and the **cos($\theta$)** leaving us with the $L_i(P,\vec{\omega_i})$ the **incident spectral radiance** function to work out. To work this out we actually have to run the same process again at the origin at were the **incident special radiance** is, recursively repeating the process until we hit a certain depth of recursion.

The **bidirectional scattering distribtion function (BSDF)** is a function that works out how much light is scattered is returned to our camera. We need to get the texture and the surface properties for this to work.

If the **BSDF** returns 0 that means that we have a non-transmissive surface. Meaning that the surface does not scatter light at all because transmissive allows light to pass through it, and light has to be absorbed to be scattered. 

With the $|\cos(\theta')|$ and the surface normal we can compute the absolute dot function which return the absolute value of the dot product between 2 vector. That's because $\vec{a} \cdot \vec{b} = ||\vec{a}||||\vec{b}||\cos(\theta)$ that's why we care about the surface normal, and if they are normalised which they should be then $\hat{a} \cdot \hat{b} = \cos(\theta)$

Finally we need to generate a new ray from the intersection point at the position of $\vec{\omega}'$ offset enough that the float point precision isn't an issue, this is to avoid a second collision with same object again. Doing this with recurision completes the approxated function.
# References
##### Main Notes
[[Introduction to Monte Carlo]]
[[The short comings of a monte carlo approach for ray tracing]]
[[How basic ray tracing works]]
[[What is the BRDF]]
[[What is the light transport equation]]
#### Source Notes
[[Physically Based Rendering - Introduction]]