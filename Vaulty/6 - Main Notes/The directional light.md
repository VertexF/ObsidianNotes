2026-06-22 14:51
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# The directional light

The point of directional light is to simulate the environment like outdoors from the sun and moon. If a light source is far enough away and bright enough turn, the light basically turns into a directional light, where the light rays are paralell. 

It's a good idea to use lux($lx$) unit for directional lights because it's easy to get these values online yourself. This also simplifies the luminance equation for [[Direct lighting theory]]
$$L_{out} = f(v,l) E_{\bot} \left< n\cdot l \right>$$
In this simplified equation the $E_{\bot}$ is the illuminance of the light source for a surface perpendicular to said light source. If this is simulating the sun $E_{\bot}$ is the illuminance of the sun for the surface perpendicular to the sun.

So here for reference this table display things in illuminance units $lx$ for a clear day in March, California

| Light                     | 10AM    | 12PM    | 5:30PM |
| ------------------------- | ------- | ------- | ------ |
| $Sky_{\bot} + Sun_{\bot}$ | 120,000 | 130,000 | 90,000 |
| $Sky_{\bot}$              | 20,000  | 25,000  | 9,000  |
| $Sun_{\bot}$              | 100,000 | 105,000 | 81,000 |
Dynamic directional lights are particularly cheap to evaluate to run at runtime.

```c
vec3 l = normalized(-lightDirection);
float NoL = clamp(dot(n, l), 0.0, 1.0);

float illuminance = lightIntensity * NoL;
vec3 luminance = (specular * Fd) * illuminance;
```
# References
##### Main Notes
[[Direct lighting theory]]
[[The point light]]
#### Source Notes
[[Filament PBR - Lighting]]
