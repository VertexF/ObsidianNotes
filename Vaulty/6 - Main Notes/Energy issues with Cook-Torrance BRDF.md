2026-06-18 15:21
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# Energy issues with Cook-Torrance BRDF

So we have two problems when it comes to energy convervation.

So with the diffuse BRDF does not account for the light that reflects at the surface that is therefor not able to participate in the diffuse scattering event. 

So when doing the specular part of the BRDF wit the Cook-Torrance version we are only account for a single light bound. This means that for high roughness a ray might bounce more than once and be reflected out. If we account for multiscattering the same ray of light might escape and reflect back to the viewer.
![[diagram_single_vs_multi_scatter.png]]
This is single scatter vs mutliscatter. 

So this is worse metallic surfaces because as you should the light is purely reflective light so you will get darker rough metals with this approach.
Here we have energy lost version
![[material_metallic_energy_loss.png]]
Here we are persering multiscattering ![[material_metallic_energy_preservation.png]]
# References
##### Main Notes
[[How to preserve energy for specular]]
[[Normal distribution function (specular D)]]
[[Geometric shadowing(specular G)]]
[[Geometric shadowing(specular G) for mobile GPUs]]
[[Fresnel (specular F)]]
#### Source Notes
[[Filament PBR - Materials]]