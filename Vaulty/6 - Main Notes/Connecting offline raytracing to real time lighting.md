2026-06-20 08:49
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# Connecting offline raytracing to real time lighting

# WARNING INCOMPLETE

This is just a note to my future self about the render equation and how real time lights and offline ray tracing are solving the same problem.

This is my favourite version of the rendering equation.
$$L_o(P, \vec{\omega_o}) = L_e(P, \vec{\omega_o}) + \int\limits_{S^2} f(P,\vec{\omega_o}, \vec{\omega_i}) L_i(P,\vec{\omega_i}) | \cos(\theta_i) | d\vec{\omega_i}$$
When we look at this function and what we are intergrating over we have the $L_e(P, \vec{\omega_o})$ **emitted light radiance** which is outside of the render equation here. 

# References
##### Main Notes
[[Direct lighting theory]]
#### Source Notes
