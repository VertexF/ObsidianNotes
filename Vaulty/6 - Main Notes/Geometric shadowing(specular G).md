2026-06-17 10:13
Status: #baby 
Tags: [[graphics theory]] [[physically based rendering]]
# Geometric shadowing(specular G)

Smith geometric shadowing is correct and exact in terms of G it looks like
$$G(v, l, a) = G_1(l,\alpha)G_1(v, \alpha)$$$G_1$ is more than 1 model and is commonly set to the GGX equation.
$$G_1(v, \alpha) = G_{GGX}(v, \alpha) = \frac{2(n \cdot v)}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}}$$
The full smith-GGX equation is.
$$G(v, l, \alpha) = \frac{2(n \cdot l)}{n \cdot l + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot l)^2}}\times\frac{2(n \cdot v)}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}}$$
We can simplify things down because of $2(n \cdot l)$ and $2(n \cdot v)$ we create another fractional function and divide by the original **G** then through cancellation we can simplify our GGX function.
$$V(v, l, \alpha) = \frac{G(v,l,\alpha)}{4(n \cdot v)(n \cdot l)} = V_1(l,a)V_1(v,a)$$
Where the denominator is just working the visibility of the specular lighthing for smith GGX. 
$$V_1(v, \alpha) = \frac{1}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}}$$
$$ = \frac{\frac{2(n \cdot l)}{n \cdot l + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot l)^2}}\times\frac{2(n \cdot v)}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}}}{\frac{4(n \cdot v)(n \cdot l)}{1}}$$
$$(\frac{2(n \cdot l)}{n \cdot l + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot l)^2}}\times\frac{2(n \cdot v)}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}})\times{\frac{1}{4(n \cdot v)(n \cdot l)}}$$
You should be able to see that the top and bottom terms of the out fractions cancels out.

Heitz notes however that taking the height of the microfacets into account correlate with masking and shadowing which leads to more accurage results. Remember $X^{+}(a)$ is the heavside function  where 1 if a>0 and 0 otherwise.
$$G(v,l,h,\alpha) = \frac{X^+(v \cdot h)X^+(l \cdot h)}{1 + \Lambda(v) + \Lambda(l)}$$
If we first look at this function which is used is the generalized form of the Smith Masking function which simplifies thing down a little if $\omega_{o}$ is the reversed vector of were the light comes from [[Light scattering at a surface]]
$$\frac{1}{1+\Lambda(\omega_{o})}$$

This $\Lambda$ function works out the height of microfacets which is
$$\Lambda(x) = \frac{-1 + \sqrt{1+\alpha^2\tan^2(\theta_x)}}{2}$$
We can convert the $\tan()$ function with some trigometric identities
$$= \frac{-1 + \sqrt{1+ \alpha^2\frac{(1 - \cos(\theta_x))}{\cos(\theta_x)}}}{2}$$
We can replace the $\cos(\theta_x)$ by $n \cdot v$ we can obtain
$$\Lambda(v) = \frac{1}{2}(\frac{\sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}}{n \cdot v} - 1)$$
From that we get the derive visibility function
$$V(v,l,\alpha) = \frac{0.5}{n \cdot l\sqrt{(n \cdot v)^2(1 - \alpha^2) + \alpha^2} +n \cdot v\sqrt{(n \cdot l)^2(1 - \alpha^2) + \alpha^2}} $$
This requires two square roots which can be expensive on some GPUs.
```c
float visibilitySmithGGXCorrelated(float NoV, float NoL, float roughness)
{
	float alpha2 = roughness * roughness;
	float GGXV = NoL * sqrt(NoV * NoV * (1 - alpha2) + alpha2);
	float GGXL = NoV * sqrt(NoL * NoL * (1 - alpha2) + alpha2);
	return 0.5 / (GGXV + GGXL);
}
```
# References
##### Main Notes
[[Normal distribution function (specular D)]]
[[Geometric shadowing(specular G) for mobile GPUs]]
[[Lambert diffuse BRDF]]
[[Fresnel (specular F)]]
#### Source Notes
[[Filament PBR - Materials]]
