2026-06-17 15:31
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# Fresnel (specular F)

This is all to do with the viewing angle of a surface and **index of refaction (IOR)**. 
![[photo_fresnel_lake.jpg]]

The steeper the angles get, the more specular the surface gets. This image shows that the steeper the viewing angle more reflective the water gets.

You also need to have the **index of refaction (IOR)** of the material. So in our note when we are talking the normal incidence, (when we are perpendicular to the surface.) We say that the amount reflected back $f_0$ can be derived from the **IOR**. The amount reflected by a grazing is $f_{90}$ which approaches 100% on smooths materials.

So Fresnel is interested in how light reflects and refact at the inteface between two different mediums. It's also concerned with the ratio reflect and transmitted energy. We have a inexpensive approximation of the Fresnel with schlick approximation.
$$F_{Schlick}(v,h,f_0,f_{90}) = f_0 + (f_{90} - f_0)(1 - v \cdot h)^5$$

The $f_0$ constant represents the specular reflectance at a normal incidence (when we are perpendicular to the surface) and is achromatic for dieletrics and chromatic for metals. In GLSL we can use `pow` and removes a few multiplications.

```c
vec3 fresnelSchlick(float v, vec3 f0, float f90)
{
	return f0 + (vec3(f90) - f0) * pow(1.0 - v, 5.0);
}
```

This due to the fact that for real world dialectrics and conductors exhibit achromatic specular reflectance, meaning that this is slightly incorrect.

If we just set the $f_{90}$ to 1 we simplify the equation to operate on scalar operations by refactoring things little.

```c
vec3 fresnelSchlick(float v, vec3 f0)
{
	float f = pow(1.0 - v, 5.0);
	return f + f0 * (1.0 - f);
}
``` 

For example when you work out the specular componenet of the BDRF

```c
vec3 specular = (D * G * F);
```
# References
##### Main Notes
[[Normal distribution function (specular D)]]
[[Geometric shadowing(specular G)]]
[[Geometric shadowing(specular G) for mobile GPUs]]
[[Fresnel (specular F)]]
[[Lambert diffuse BRDF]]
[[What is ambient light]]
#### Source Notes
[[Filament PBR - Materials]]