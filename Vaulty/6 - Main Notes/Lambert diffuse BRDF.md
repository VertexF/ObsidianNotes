2026-06-17 16:04
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# Lambert diffuse BRDF

The diffuse term is a lambertian function and the diffuse term of the BRDF become
$$f_d(v,l) = \frac{\sigma}{\pi} \frac{1}{| n \cdot v | | n \cdot l |}
\int_\Omega D(m,\alpha) G(v,l,m) (v \cdot m) (l \cdot m) dm$$
We are going to talk the most basic Lambertian function that assume we have a uniform diffuse respone over microfacets.
$$f_d(v,l) = \frac{\sigma}{\pi}$$
```c
float fdLambert()
{
	return 1.0 / PI;
}

void main()
{
	vec3 fd = diffuseColour * fdLambert();
}
```

This is extremely efficient when it comes to delivers result close enough to more complex models. 
# References
##### Main Notes
[[Disney's diffuse BRDF]]
[[Normal distribution function (specular D)]]
[[Geometric shadowing(specular G)]]
[[Geometric shadowing(specular G) for mobile GPUs]]
[[Fresnel (specular F)]]
#### Source Notes
[[Filament PBR - Materials]]
