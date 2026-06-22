2026-06-17 10:15
Status: #baby 
Tags: [[graphics theory]]
# Geometric shadowing(specular G) for mobile GPUs

Instead of [[Geometric shadowing(specular G)]] There are two optimisations that you can make for speed while reducing the accuracy if you care about the square roots. This is only really needed if you're targetting mobile GPUs.
$$V(v,l,\alpha) = \frac{0.5}{n \cdot l (n \cdot v (1 - \alpha) + \alpha) + n \cdot v (n \cdot l (1 - \alpha) + \alpha)}$$
This assumes that all the turns of the square root within the range of $[0, 1]$ 

You can also do the same with a lerp function
$$V(v,l,\alpha) = \frac{0.5}{lerp(2 (n \cdot l) (n \cdot v ), n \cdot l  + n \cdot v , \alpha)}$$
So the faster version of this in GLSL. 
```c
float visibilitySmithGGXCorrelatedFast(float NoV, float NoL, float roughness)
{
	float alpha = roughness;
	float GGXV = NoL * (NoV * (1 - alpha) + alpha);
	float GGXL = NoV * (NoL * (1 - alpha) + alpha);
	return 0.5 / (GGXV + GGXL);
}
```
# References
##### Main Notes
[[Geometric shadowing(specular G)]]
[[Normal distribution function (specular D)]]
[[Fresnel (specular F)]]
[[Lambert diffuse BRDF]]
#### Source Notes
[[Filament PBR - Materials]]