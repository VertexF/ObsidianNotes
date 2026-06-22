# Reference https://google.github.io/filament/main/filament.html#lighting

It's a good to have lights that have coherent lighthing units that all follow the same system. You have two different lighting types 

Direct lighting and indirect lighting.

| Photometric terms   | Notation   | Unit                             |
| ------------------- | ---------- | -------------------------------- |
| Luminous power      | $\Phi$     | Lumen($lm$)                      |
| Luminous intensity  | $I$        | Candela($cd$) or $\frac{lm}{sr}$ |
| Illuminance         | $E$        | Lux($lx$) or $\frac{lm}{m^{2}}$  |
| Lumiance            | $L$        | Nit ($nt$) or $\frac{cd}{m^2}$   |
| Radiant power       | $\Phi_{e}$ | Watt($W$)                        |
| Luminous efficacy   | $\eta$     | Lumens per watt ($\frac{lm}{W}$) |
| Luminous efficiency | $V$        | Percentage $(\%)$                |
Illuminance is the total luminous flux incident on a surface per unit area. That is why we mesure this with a $\frac{lm}{m^{2}}$ because it's per unit area.

Lumiance is a measure of the luminous intensity per unit area of light travelling in a given direction. That's also why we are mesuring $\frac{cd}{m^2}$ per unit area vs travelled.

Candela $cd$ is a standard unit that cares about the luminous power over a solid angle from a given light. A solid angle can be thought of as a cone were the apex of the cone starts at the light source.

![[Angle_solide_coordonnees.svg.png]]

So it's good idea to make sure your lights map to real world light units. Here table of common lights and the unit you should try to use. 

| Light type             | Unit                                   |
| ---------------------- | -------------------------------------- |
| Directional light      | Illuminance ($lx$ or $\frac{lm}{m^2}$) |
| Point light            | Luminous power ($lm$)                  |
| Spot                   | Luminous power ($lm$)                  |
| Photometric light      | Luminous intensity ($cd$)              |
| Mask photometric light | Luminous power ($lm$)                  |
| Area light             | Luminous power ($lm$)                  |
| Image based light      | Luminance ($\frac{cd}{m^2}$)           |
So if want to use watts to work with light brightness you can use this formular to go from lumen over watts to just lumens.
$$\Phi = \Phi_e \eta$$
So $\eta$ is luminous efficacy of the light expressed in lumens per watt. The maximum possible luminous efficacy is $683\frac{lm}{W}$ We can also just use $V$ also called liminous coefficient. As this is a power using the number of photons per watt.
$$\Phi = \Phi_e 683 \times V$$
Look online to learn specific lighthing luminous efficacy or the luminous efficiency if you're using a coherent lighting system.
##### Measuring basics
One advantage of using phyiscal light units is that we have ability validate our equation.

When we are talking about measuring **Luminace** we are measuring candela per meter squared which intersects on a plane with a stridian [[Lighting unit basics]] of 5 degree in picture below.

![[measuring-tool-for-lumance-2.png]]

So with this we can work out the intensity of light when using the inverse of the inverse square law of light, if we know the distance.
$$I = E \cdot d^2$$

This might not be as important as just matching the real world lights to get nice accurate lighthing.
##### Direct lighting
Even if you have the measured carefully all the lighthing units. You still need able to convert things to make sense for the different light equations. When we are computing direct lighting we are computing the luminance values in our shader for any given point. The final light of that point depends on illumiance $E$ and the **BRDF** or **BSDF**.
$$L_{out} = f(v,l)E$$
##### Directional lights
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
##### Punctual lights
Spot and punctual lights don't follow the inverse square and can be truly infinitestimally small.

If you care about performance it's good to use infinitesimally small punctual light when possible.

We can solve the first issue by making sure that the inverse square law has the term $E$ in the equation $L_{out} = f(v,l)E$ using that we express this idea with this equation.
$$E = L_{in} \left< n\cdot l \right> = \frac{I}{d^2} \left< n \cdot l \right>$$
The difference between a point and a spot light lies in how $E$ is computed, and in particular how luminous intensity $I$ is computed from the luminous power $\Phi$

Meaning when we are looking that the equation above if you're punctual light $\frac{I}{d^2}$ is the starting point before you begin writing anything. Note this value is **NOT** luminance ($L$) as that would per unit area, and not distance squared.
##### Point lights
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
    //Remember that a point light is defined only by it's position in space.
    float distanceSquared = pow(distance, 2.0);
    float distanceOverRange = min(1.0 - pow(distance / lightRange, 4.0), 1.0);
    
    return lumens * max(distanceOverRange, 0.0) / distanceSquared;
}
```
##### Spot lights
A spot light is defined by a position in space, a direction vector and two cone angles, $\theta_{inner}$ and $\theta_{outer}$ The two angle are used to define the angular falloff attenuation of the spot light. So the light equation must take into account both the inverse square law and these two angles to properly evalute the luminance attenuation.
![[diagram_spot_light.png]]

Just like with [[The spot light]] you can work this equation out with the $\theta_{outer}$ the outer angle of the spot light's cone range in the domain of $[0, \pi]$ 
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

To correctly work with spot lights, you need to surround your spot in a point light hence the sphere of influence. [[Attuation function]]
##### Attenuation function
We can't use the standard mathematical equation, this is because it's impractical implement.
1) The division by the square distance can lead to divides by 0, when an object touches the light source.
2) The influence sphere of each light is infinite. ($\frac{I}{d^2}$ is asymptotic, it never reaches 0) which mean that to correctly shade a pixel we need to evaluate every light in the world.
The first issue that we are have can be solved by making the assumption that we are have an area light with a small area of 1cm.
$$E = \frac{I}{max(d^2, {0.01}^2)}$$
The second issue we need to introduce a sphere influence each light. This allows us to use the debug sphere to show the influence. We can also cull lights more aggrestively using this extra piece of information.  With a influence sphere you can manually tweak the radius of the light which might help out artists.

Mathematically, the illuminance of a light should smoothly reach zero at the limit definied by a influence radius. In this [paper](Brian Karis, 2013. Real Shading in Unreal Engine 4. [https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf](https://blog.selfshadow.com/publications/s2013-shading-course/karis/s2013_pbs_epic_notes_v2.pdf)) we take the inverse square function and we use a function that allow the ligths influence to remain unaffected. We have this in with this equation were the radius $r$
$$E = \frac{I}{max(d^2, {0.01}^2)} \left< 1 - \frac{d^4}{r^4} \right>^2$$
Note this equation is purely for the sphere of influence and it's nothing to do with a spot light itself. 