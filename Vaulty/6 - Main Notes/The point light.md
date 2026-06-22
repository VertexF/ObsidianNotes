2026-06-22 14:41
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# The point light

The point is defined only by it's position in space.
![[diagram_point_light.png]]

The luminous power (lumens $\Phi$) of point light is calculated by intergating the luminous intensity ($I$ Candela($cd$) or $\frac{lm}{sr}$) over the light's solid angle, as show here. 
$$\Phi = \int_{\Omega} I dl = \int_{0}^{2\pi} \int_{0}^{\pi} I d\theta d\phi = 4 \pi I$$
$$I = \frac{\Phi}{4 \pi}$$
In this case $d$ is distance. When we are intergating over a steradian, it's a double intergral because of the third dimentionality of steradians. We can simply derive the value over set steradian and use that instead of doing an intergation. 

Remember here we are trying to add lighthing to our BRDF by multiplying [[Punctual lights theory|luminous intensity divided by (distance sqaured)]] $d^2$. So with that in mind we throwing in our lighting equation + BRDF + the direction of the light we get this.

$$L_{out} = f(v,l)\frac{\Phi}{4\pi d^2} \left< n \cdot l \right>$$

How this works is that we aren't multiplying $\frac{I}{d^2}$ by $\frac{\Phi}{4 \pi}$ we are saying the equation needs to follow the pattern of $$E = L_{in} \left< n\cdot l \right> = \frac{I}{d^2} \left< n \cdot l \right>$$
So when we working out the luminous intensity $I = \frac{\Phi}{4 \pi}$ we are just missing adding the distance to the equation.

An implementation of a point light might look like this
```c
float pointLight(float lightRange, float lumens, float distance)
{
	float distanceSquare = dot(posToLight, posToLight);
	float factor = distanceSquare * lightInvRadius * lightInvRadius;
	float smoothFactor = max(1.0 - factor * factor, 0.0);
	return (smoothFactor * smoothFactor) / max(distanceSquare, 1e-4);
}
```
# References
##### Main Notes
[[Lighting unit basics]]
[[Punctual lights theory]]
[[The spot light]]
[[Attenuation function]]
#### Source Notes
[[Filament PBR - Lighting]]
