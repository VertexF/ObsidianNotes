2026-06-16 15:56
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# Normal distribution function (specular D)

In the **normal distribution function (NDF)** long-tailed is a better fit for real world surfaces. The GGX distrubition with long-tailed falloff and short peak in the highlights, with a simple equation suitable for real-time implementations. The popular model is the **Trowbridge-Reitz** distribution.
$$D_{GGX}(h, \alpha) = \frac{\alpha^2}{\pi((n\cdot h)^2(\alpha^2 - 1) + 1)^2}$$
```c
float D_GGX(float NoH, float roughness)
{
	float a = NoH * roughness;
	float k = roughness / (1.0 - NoH * NoH + a * a);
	return k * k * (1.0 / PI);
}
```
# References
##### Main Notes
[[What is specular BRDF]]
[[Geometric shadowing(specular G)]]
[[Lambert diffuse BRDF]]
[[Fresnel (specular F)]]
#### Source Notes
[[Filament PBR - Materials]]