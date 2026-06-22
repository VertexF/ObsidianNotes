2026-06-17 16:14
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# Disney's diffuse BRDF

However, diffuse part would ideally be coherent with the specular term and take into account surface roughness. So the Disney diffuse BRDF and Oren-Nayar take roughness into. So you can use better diffuse BRDF which improves the accuracy of the grazing angles by retro-reflections however, image-based and spherical harmonics based techniques are harder to implement.

Here is the Disney's diffuse BRDF expressed like:
$$fd(v,l) = \frac{\sigma}{\pi} F_{schlick}(n,l,1,f_{90}) F_{schlick}(n,v,1,f_{90})$$
Where:
$$f_{90} = 0.5 + 2 \cdot \alpha\cos^2(\theta_d)$$
```c
vec3 fresnelSchlick(float v, vec3 f0, float f90)
{
	return f0 + (vec3(f90) - f0) * pow(1.0 - v, 5.0);
}

float fdBurley(float NoV, float NoL, float LoH, float roughness)
{
	float f90 = 0.5 + 2.0 * roughness * LoH * LoH;
	float lightScatter = fresnelSchlick(NoL, 1.0, f90);
	float viewScatter = fresnelSchlick(NoV, 1.0, f90);
	return lightScatter * viewScatter * (1.0 / PI);
}
```

![[diagram_lambert_vs_disney.png]]
The sound one is Disney and you have nice grazing shadows in your sphere. The Disney diffuse BRDF is not energy conversing.
# References
##### Main Notes
[[Lambert diffuse BRDF]]
#### Source Notes
[[Filament PBR - Materials]]