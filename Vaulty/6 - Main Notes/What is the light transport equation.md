2026-06-12 09:12
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# What is the light transport equation

The amount of light that reaches the camera from a point is given by the sum of light emitted by an object and a light source, and the mount of light reflected. This idea is formalised in the **light transport equation** aka **rendering equation**

This measures light with respect to radiance, a radiometric unit. The output for the equation is $L_o(P, \hat{\omega_o})$ which states that from point **P** and direction $\vec{\omega_o}$ is emitted radiance at the point of direction $L_e(P, \hat{\omega_o})$ which is the **emitted light radiance** function.

Next we do this over all the incident radiance from all direction over a hemasphere from point **P** scaled by the [[What is the BRDF|BRDF]] $f_r(P,\hat{\omega_o}, \hat{\omega_i})$ and the cosine term:

$$L_o(P, \hat{\omega_o}) = L_e(P, \hat{\omega_o}) + \int\limits_{S^2} f_r(P,\hat{\omega_o}, \hat{\omega_i}) L_i(P,\hat{\omega_i}) | \cos(\theta_i) | d\hat{\omega_i}$$
You can't solve this analytically apart from in the most simplest of scenes, you either must simplify the assumptions or use numerical integration technique.

Whitted's ray-tracing algorithm simplifies this integral by incoming light from most direction only in $L_i(P,\hat{\omega_i})$ (**light indicident radiance**) for direction to light sources and directions of perfect reflections and refractions. In short this turns the integral into a small sum over a small number of directions.

You can just randomly sample but that's very slow and you'll need to improve your algorithms to speed it up.
# References
##### Main Notes
[[Indirect light transport for rays]]
[[Light distribution]]
[[The theory of handling more than 1 light]]
#### Source Notes
[[Physically Based Rendering - Introduction]]