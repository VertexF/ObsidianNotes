2026-06-18 17:03
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# PBR parameter types

Disney's material model describes to BRDF well but it has FAR to many parameters which makes it impractical for real-time rendering.

Generally you would want to have these for your parameters for your PBR.

| Parameters     | Definition                                                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `baseColour`   | This is the diffuse albedo fir non-metallic surface and the specular colour for metallic surfaces.                                   |
| `metalness`    | Whether a surface appears to be a dielectric to conductor over the range $[0, 1]$                                                    |
| `roughness`    | How rough something is from smooth to rough $[0, 1]$ Smooth surfaces exhibit sharp reflections.                                      |
| $\reflectance$ | Fresnel reflectance at a normal incidence for dielectric surface. This replaces an explicit index of refaction.                      |
| `emissive`     | This is an additional diffuse to simulate emssive surfaces. This is helpful in HDR with bloom.                                       |
| $\ao$          | Ambient occlusion define how much of ambient light accessible at a surface point. It is a per-pixel shadowing factor from 0.0 to 1.0 |

Here we have the ranged of the different parameters, note this per-pixel in a fragment shader.

| Parameters          | Definition                                                                                                            |
| ------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `baseColour`        | Is a linear RGB value because we are sampling over a standard texture                                                 |
| `metalness`         | This is a scalar value because we sampling over a texture and we include a factor + the **blue** value of a texture.  |
| `roughness`         | This is a scalar value because we sampling over a texture and we include a factor + the **green** value of a texture. |
| `reflection`        | is a scalar value I don't have in my engine.                                                                          |
| `emissive`          | Is another texture like the base but also adds an exposure compensation.                                              |
| `ambient occlusion` | is a scalar value I don't have my engine but hardcode a `0.01 * baseColour` for that.                                 |

# References
##### Main Notes
#### Source Notes
[[Filament PBR - Materials]]