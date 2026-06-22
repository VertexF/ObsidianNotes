# Reference https://google.github.io/filament/main/filament.html#material-system


| Symbol       | Definition                                                 |
| ------------ | ---------------------------------------------------------- |
| $v$          | View unit vector                                           |
| $l$          | Incident light uint vector                                 |
| $n$          | Surface normal unit vector                                 |
| $h$          | Half unit vector between $l$ and $v$                       |
| $f$          | BRDF                                                       |
| $f_d$        | Diffuse component of a BRDF                                |
| $f_r$        | Specular component of a BRDF                               |
| $\alpha$     | Roughness, remapped from using input `perceptualRoughness` |
| $\sigma$     | Diffuse reflectance                                        |
| $\Omega$     | Spherical domain                                           |
| $f_0$        | Reflectance at normal incidence                            |
| $f_{90}$     | Reflectance at grazing angle                               |
| $X^{+}(a)$   | Heaviside function (1 if a>0 and 0 otherwise)              |
| $n_{ior}$    | Index of refraction (IOR) of an interface                  |
| $<n\cdot l>$ | Dot product clamped to $[0..1]$                            |
| $<a>$        | Saturated value (clamped to $[0..1]$)                      |
##### Standard model
So the **BSDF (Bidirectional Scattering Distribution Function)** is made up for 2 models. The 
**BRDF (Bidirectional Scattering Distribution Function), BRDF (Bidirectional Reflectance Distribution Function) and the BTDF (Bidirectional Transmittance Function)**.

So BRDF is made up of two component $f_d$ and $f_r$  this would look something like this with BRDF, note the surface normal $n$ and the $l$

![[diagram_fr_fd.png]]

We would write the BRDF as $f(v, l) = f_d(v, l) + f_r(v, l)$ we of course would need to intergate over a hemisphere to solve this under rendering equation.

A microfacet BRDF this a good BRDF for our purposes. This talks about things aren't smooth at the micro level. They are made up of randonly aligned planar surface fragments called microfacets. Only the microfacet whose normal is oriented halfway between the light direction and view direction will reflect visible light. ![[diagram_macrosurface.png]]

Not all microfacets witha properly oriented normal will contribute reflect because of shadow and masking.![[diagram_shadowing_masking.png]]

With increased roughness the specular highlight blur as the light is more reflected scattered away from the camera. ![[diagram_roughness.png]]

So here the render equation again but with a BRDF and the $\cos()$ as a dot product instead.
$$f_x(v,l) = \frac{1}{|n\cdot v||n\cdot l|} \int_\Omega D(m,\alpha)G(v,m,l)f_m(v,l,m)(v\cdot m)(l \cdot m)dm$$
The term $D$ models the distribution of the microfacets. This term is also called the **Normal Distribution Function (NDF)** The term play a primordial role in the apperance of surfaces, as you can see in the image above.

The term $G$ models the visibility is the occluded or shadow masked part of the microfacets.

To simplify things we make that when a light ray hit a surface we assume that fragment were it lands is also flat, meaning that a single light ray fires from that surface. 
##### Dielectrics and conductors
Dielectric are non-metallic surfaces and conductors are metallic. When it comes to purely metallic materials they do not do any sub-surface scattering which means there is no diffuse component. Meaning we need to make sure we treat these differently from each other.
##### Specular BRDF
So using the Cook-Torrance approximation of microfacet model integration with with Fresnel law for our BRDF. 
$$f_r(v, l) = \frac{D(h,\alpha)G(v,l,\alpha)F(v,h,f_0)}{4(n \cdot v)(n \cdot l)}$$
We must approximate for the three terms $D, G$ and $F$. If you use [this article](https://graphicrants.blogspot.com/2013/08/specular-brdf-reference.html) you'll be able to see a list of equation that approximate different parts of with the Cook-Torrance specular **BRDF**.
##### Normal distribution function (specular D)
In the **normal distribution function (NDF)** long-tailed is a better fir for real world surfaces. The GGX distrubition with long-tailed falloff and short peak in the highlights, with a simple formulation suitable for real-time implementations. The popular model is the **Trowbridge-Reitz** distribution.
$$D_{GGX}(h, \alpha) = \frac{\alpha^2}{\pi((n\cdot h)^2(\alpha^2 - 1) + 1)^2}$$
```c
float D_GGX(float NoH, float roughness)
{
	float a = NoH * roughness;
	float k = roughness / (1.0 - NoH * NoH + a * a);
	return k * k * (1.0 / PI);
}
```
##### Geometric shadowing(specular G)
Smith geometric shadowing is correct and exact in terms of G it looks like
$$G(v, l, a) = G_1(l,\alpha)G_1(v, \alpha)$$$G_1$ is more than 1 model and is commonly set to the GGX equation.
$$G_1(v, \alpha) = G_{GXX}(v, \alpha) = \frac{2(n \cdot v)}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}}$$
The full smith-GGX equation is.
$$G(v, l, \alpha) = \frac{2(n \cdot l)}{n \cdot l + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot l)^2}}\times\frac{2(n \cdot v)}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}}$$
We can simplify things down because of $2(n \cdot l)$ and $2(n \cdot v)$ we create another fractional function and divide by the original **G** then through cancellation we can simplify our GGX function.
$$V(v, l, \alpha) = \frac{G(v,l,\alpha)}{4(n \cdot v)(n \cdot l)} = V_1(l,a)V_1(v,a)$$
Where the denominator is just working the visibility of the specular lighthing for smith GGX. 
$$V_1(v, \alpha) = \frac{1}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}}$$
$$ = \frac{\frac{2(n \cdot l)}{n \cdot l + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot l)^2}}\times\frac{2(n \cdot v)}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}}}{\frac{4(n \cdot v)(n \cdot l)}{1}}$$
$$(\frac{2(n \cdot l)}{n \cdot l + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot l)^2}}\times\frac{2(n \cdot v)}{n \cdot v + \sqrt{\alpha^2 + (1 - \alpha^2)(n \cdot v)^2}})\times{\frac{1}{4(n \cdot v)(n \cdot l)}}$$
You should be able to see that the top and bottome terms of the out fractions cancels out.

Heitz notes however that taking the height of the microfacets into account correlate with masking and shadowing which leads to more accurage results. Remember $X^{+}(a)$ is the heavside function  where 1 if a>0 and 0 otherwise.
$$G(v,l,h,\alpha) = \frac{X^+(v \cdot h)X^+(l \cdot h)}{1 + \Lambda(v) + \Lambda(l)}$$
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

##### Geometric shadowing(specular G) for mobile GPUs
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

##### Fresnel (specular F)
This is all to do with the viewing angle of a surface and **index of refaction (IOR)**. 
![[photo_fresnel_lake.jpg]]

The steeper the angles get, the more specular the surface gets. This image shows that the steeper the viewing angle more reflective the water gets.

You also need to have the **index of refaction (IOR)** of the material. So in our note when we are talking the normal incidence, (when we are perpendicular to the surface.) We say that the amount reflected back $f_0$ can be derived from the **IOR**. The amount reflected by a grazing is $f_{90}$ which approaches 100% on smooths materials.

So Fresnel is interested in how light reflects and refact at the inteface between two different media. It also concerned with the ratio reflect and transmitted energy. We have a inexpensive approximation of the Fresnel
$$F_{Schlick}(v,h,f_0,f_{90}) = f_0 + (f_{90} - f_0)(1 - v \cdot h)^5$$

The $f_0$ constant represents the specular reflectance at a normal incidence (when we are perpendicular to the surface) and is achromatic for dieletrics and chromatic for metals. So the actual value refers to **IOR**. In GLSL we can use `pow` and removes a few multiplications.

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

When you are using the Fresnel Schlick function for the Cook Torrance specular BRDF, you'll want to make sure that values are 0.0001 to avoid pure black. 

For example when you work out the specular componenet of the BDRF
```c
vec3 specular = (D * G * F) / (4.0 * clamp(dot(N, V), 0.0, 1.0) * NoL + 0.00001);
```

Ambient is the definition of if the environemnt is pure lightless or habe some basic light that feeds the environment. Since your're not doing ray-traced light. It's a very good idea to suggest to boost your albedo.
##### Diffuse BRDF
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
##### Disney's diffuse BRDF
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
##### Energy issues with Cook-Torrance BRDF
So we have two problems when it comes to energy convervation.

So with the diffuse BRDF does not account for the light that reflects at the surface that is therefor not able to participate in the diffuse scattering event. 

So when doing the specular part of the BRDF wit the Cook-Torrance version we are only account for a single light bound. This means that for high roughness a ray might bounce more than once and be reflected out. If we account for multiscattering the same ray of light might escape and reflect back to the viewer.
![[diagram_single_vs_multi_scatter.png]]
This is single scatter vs mutliscatter. 

So this is worse metallic surfaces because as you should the light is purely reflective light so you will get darker rough metals with this approach.
Here we have energy lost version
![[material_metallic_energy_loss.png]]
Here we are persering multiscattering ![[material_metallic_energy_preservation.png]]

##### How to preserve energy for specular
We can use a white furnace, a uniform lighting environment set to pure white, to validate the energy preservation property of a BRDF. When you do this pure reflective metnals are indistinguishable from the background, no matter the roughness of said surface. 

![[material_furnace_energy_loss.png]]
This should what a surface looks with specular BRDF presented in [[Energy issues with Cook-Torrance BRDF]]  

Mutliple-scatter microfacets BRDFs are discussed in depth in [Heitz2016](https://dl.acm.org/doi/10.1145/2897824.2925943) Sadly this only presents a stochatic evalution of multiscattering BRDF. Therefore this is not suitable for real-time rendering. Kulla and Conty present a different approach in [Kulla17](https://fpsunflower.github.io/ckulla/data/s2017_pbs_imageworks_slides_v2.pdf) Their idea is to add an energy compensation term a an additional BRDF lobe should in this equation.
$$f_{ms}(l,v) = \frac{(1 - E(l)) (1 - E(v)) F_{avg}^2 E_{avg}}{\pi (1 - E_{avg}) (1 - F_{avg}(1 - E_{avg}))}$$
$E$ is where the direction albedo of the specular BRDF $f_r$ with $f_0$ set to 1. Which I believe is the colour of the specular light reflected back.
$$E(l) = \int_{\Omega} f(l,v) (n\cdot v) dv$$
The term $E_{avg}$ is the consine-weighted average of $E$
$$E_{avg} = 2 \int_0^1 E(\mu) \mu d\mu$$
$F_{avg}$ is the cosine-weighted average of the Fresnel term:
$$F_{avg} = 2 \int_0^1 F(\mu) \mu d\mu$$
Both terms $E$ and $E_{avg}$ can be precomputed stored in lookup tables. $F_{avg}$ can simplified when the Schlick approximation
$$F_{avg} = \frac{1 + 20f_{0}}{21}$$
The new lobe is combined $f_r$ with the orignal single scattering lobe, which is the original specular BRDF $f_{ss}$ and the multi-scatter lobe $f_{ms}$ which gives us
$$f_{r}(l,v) = f_{ss}(l,v) + f_{ms}(l,v)$$
So according to Emmanuel Turquin, Lagarde and Golubev makes the observation that the $F_{avg}$ can be just $f_{0}$ with the addtion of applying energy compensation by adding scaled GGX specular
$$f_{ms}(l,v) = f_{0} \frac{1 - E(l)}{E(l)} f_{ss}(l,v)$$
So we of course can pre-compute $E(l)$ via a lookup table it can also be shared with image-based lighting for pre-integration. So multiscattering energy compensations equation is
$$f_r(l,v) = f_{ss}(l,v) + f_{0} \left( \frac{1}{r} - 1 \right) f_{ss}(l,v)$$
Where $r$ is defined as
$$r = \int_{\Omega} D(l,v) V(l,v) \left< n \cdot l\right> dl$$
So to simply the $r$ with this integral we can have DFG lookup table we simplify things to be like this
```c
vec3 energyCompensation = 1.0 + f0 (1.0 / dfg.y - 1.0);
//Scale the specular lob to account for multiscatter.
F *= energyCompensation;
```
#### NOTE: - I have not yet learnt how to precompute DFG lookup tables.

##### Parameterization
Disney's material model describes to BRDF well but it has FAR to many parameters which makes it impractical for real-time rendering.

Generally you would want to have these for your parameters for your PBR.

| Parameters     | Definition                                                                                                                           |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `baseColour`   | This is the diffuse albedo fir non-metallic surface and the specular colour for metallic surfaces.                                   |
| `metalness`    | Whether a surface appears to be a dielectric to conductor over the range $[0, 1]$                                                    |
| `roughness`    | How rough something is from smooth to rough $[0, 1]$ Smooth surfaces exhibit sharp reflections.                                      |
| $\reflectance$ | Fresnel reflectance at a normal incidence for dielectric surface. This replaces an explicit index of refaction.                      |
| `emissive`     | This is an additional diffuse to simulate emssive surfaces. This is helpful in HDR with bloom.                                       |
| $\ao$          | Ambient occlusion define how much of ambient light accessible at a surface point. It is a per-pixel shadowing factor from 0.0 to 1.0 |

Here we have the ranged of the different parameters, note this per-pixel in a fragment shader.

| Parameters          | Definition                                                                                                            |
| ------------------- | --------------------------------------------------------------------------------------------------------------------- |
| `baseColour`        | Is a linear RGB value because we are sampling over a standard texture                                                 |
| `metalness`         | This is a scalar value because we sampling over a texture and we include a factor + the **blue** value of a texture.  |
| `roughness`         | This is a scalar value because we sampling over a texture and we include a factor + the **green** value of a texture. |
| `reflection`        | is a scalar value I don't have in my engine.                                                                          |
| `emissive`          | Is another texture like the base but also adds an exposure compensation.                                              |
| `ambient occlusion` | is a scalar value I don't have my engine but hardcode a `0.01 * baseColour` for that.                                 |

##### Reflectance remapping
When working out the metallicness of meterials. You can't just use the base colour because of this with $f_{0}$. You need to correctly remap the base to be used in both conductors and specular light and diffuse for dielectrics.

```c
vec3 f0 = mix(vec3(0.04), baseColour.rgb, metalness);
//OR
vec3 diffuseColor = (1.0 - metallic) * baseColor.rgb;
vec3 f0 = diffuseColor * vec3(0.04);
```
