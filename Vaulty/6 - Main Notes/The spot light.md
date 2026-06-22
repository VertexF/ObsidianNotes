2026-06-22 14:45
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# The spot light

A spot light is defined by a position in space, a direction vector and two cone angles, $\theta_{inner}$ and $\theta_{outer}$ The two angle are used to define the angular falloff attenuation of the spot light. So the light equation must take into account both the inverse square law and these two angles to properly evalute the luminance attenuation.
![[diagram_spot_light.png]]

Just like with [[The point light]] you can work this equation out with the $\theta_{outer}$ the outer angle of the spot light's cone range in the domain of $[0, \pi]$ 
$$\Phi = \int_{\Omega} I dl = \int_{0}^{2\pi} \int_{0}^{\theta_{outer}} I d\theta d\phi = 2 \pi (1 - cos\frac{\theta_{outer}}{2})I$$
$$I = \frac{\Phi}{2 \pi (1 - cos\frac{\theta_{outer}}{2})}$$
So with equation as you decrease the outer cone aperture, the illumination level increases. ![[screenshot_spot_light_focused.png]]

To fix this you'll want to add this to the equation
$$\Phi = \pi I$$
$$I = \frac{\Phi}{\pi}$$
This equation can also be considered physically based if the spot's reflector is replaced with a matte, diffuse mask that absorbs light perfectly.

There are two equations one with a light absorber and one with a light reflector.
1) This is with a light absorber.
$$L_{out} = f(v,l) \frac{\Phi}{\pi d^2} \left< n\cdot l\right> \lambda(l)$$
2) This is with a light reflector.
$$L_{out} = f(v,l) \frac{\Phi}{2 \pi (1 - cos\frac{\theta_{outer}}{2}) d^2} \left< n \cdot l \right> \lambda(l)$$
The $\lambda(l)$ in equations above is the spot's angle attenuation factor
$$\lambda(l) = \frac{l \cdot spotDirection - cos\theta_{outer}}{cos\theta_{inner} - cos\theta_{outer}}$$

```c
float getSquareFalloffAttenuation(vec3 posToLight, float lightInvRadius)
{
	//This d factor in 
	float distanceSquare = dot(posToLight, posToLight);
	float factor = distanceSquare * lightInvRadius * lightInvRadius;
	float smoothFactor = max(1.0 - factor * factor, 0.0);
	return (smoothFactor * smoothFactor) / max(distanceSquare, 1e-4);
}

float getSpotAngleAttenuation(vec3 l, vec3 lightDir, float innerAngle, float outerAngle)
{
	//The scale and offset computations can be done CPU-side.
	float cosOuter = cos(outerAngle);
	float spotScale = 1.0 / max(cos(innerAngle) - cosOuter, 1e-4);
	float spotOffset = -cosOuter * spotScale;
	
	float cd = dot(normalize(-lightDir), l);
	float attenuation = clamp(cd * spotScale + spotOffset, 0.0, 1.0);
	return attenuation * attenuation;
}

//In your engine main function.
vec3 evaluatePunctualLight()
{
	vec3 l = normalize(posToLight);
	float NoL = clamp(dot(n, l), 0.0, 1.0);
	vec3 posToLight = lightPosition - worldPosition;
	
	float attenuation;
	attenuation = getSquareFalloffAttenuation(posToLight, lightInvRadius);
	attenuation *= getSpotAngleAttenuation(l, lightDir, innerAngle, outerAngle);
	
	vec3 luminance = ((specular * Fd) * lightIntensity * attenuation * NoL) * vec3(0.4, 0.8, 0.1);
    return luminance;
}
```

To correctly work with spot lights, you need to surround your spot in a point light hence the [[Attenuation function|sphere of influence]].
# References
##### Main Notes
[[Lighting unit basics]]
[[The point light]]
[[Punctual lights theory]]
[[Attenuation function]]
#### Source Notes
[[Filament PBR - Lighting]]