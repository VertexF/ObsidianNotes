# Reference https://www.pbr-book.org/4ed/Introduction

##### Basic of ray tracing
You first need to be able to figure out were a ray interact on a given object. We need to know the properties of a given object, such as the materials and normals etc. So ray traces have the ability testing the intersection of rays with mulit object and returning the colost intersection along ray.

We also need to be able work out the distribution of light and location of lights. This include how the ray distribute their energy throughout a scene.

So we need to be able to figure if an ray is block by something we do this by making sure that the position of the ray comes from a light, hits a object we are interested. We only return the shortest ray.

We also need to make sure that know how the light was scattered on a surface, typically this is model surfaces are paramtertised so they can simulate varity of appearance.

You might need to worry about what the ray passes through such as smoke, fog and Earth atmosphore.
##### Camera and film
The **eye** of a viewing volume is the place at which the viewing volume begins. When it comes to what we can see, we are only interested in the light that comes from a plane, which is what we are looking.

The camera's job is to generate a ray that travels and hits the plane in away that contributes to the light of an image. Using a origin point and direction vector this is easy to do. We might have to do more if the lenses of our camera more complex than a pin-hole camera.

Light comes frm a ray that hits camera has different energy levels, and just like a camera sensor we are only interested in RGB values to recreate the colour. 
##### Ray-Object Intersections
The renderer need to figure out were and when the ray if at all hit something from the camera. We are interested in what the ray hits first. This is to get the point at which we need to simulate light.

We can use the parametric form of a ray for this. If **r** is a a ray and **d** is the direction vector and **o** is the origin at which the ray starts.

$$r(t) = o + t\hat{d}$$
If we use the **t** parameter which goes from $[0, \infty)$ we can travel down an array to find the point given value **t**.

We can easily find the interaction with a function$F(x, y, z) = 0$ So we substitute the ray equation into the implicit equation and make a function whose only paramter is **t**. We solve this function for **t** and substitute the smallest possible root into the ray equation to find the point.

So if we have a sphere defined like this. 
$$x^2 + y^2 + z^2 - r^2 = 0$$
We can throw in our ray equetion like this

$$(o_x + t\hat{d}_x) + (o_y + t\hat{d}_y) + (o_z + t\hat{d}_z) - r^2 $$
This give us a quadratic equation in **t**. If there are no real roots, the ray missed. If there are real roots, then we just take the smallest positive integer of **t** to get the interaction point.

So imagine the equation above, now we cast a ray with the equation $$r(t) = o + t\hat{d}$$
Over the quadratic 
$$(o_x + t\hat{d}_x)^2 + (o_y + t\hat{d}_y)^2 + (o_z + t\hat{d}_z)^2 - r^2 $$

and we do this until we have a real root, while increase **t** by one. Once we have a real root, we have an interaction.
##### Properties for the ray tracer
Just doing [[Ray-object Intersections]] for a given is not enough. We need to pass the properies of a given surface to the next part of the ray tracing algorithm. To shader the point we need to pass the geometric information as well. 

We **ALWAYS** need to to pass the surface normal $\hat{n}$ to later parts of the algorithm. So need to worry about bother partial derivatives of position and surface normal with respect to local parameters of the surface. You can however just pass $\hat{n}$ for a simple ray tracer.
##### The general approach to ray tracing
The brute force approach would be to past rays out for each object in a scene until we are done. This is extremely slow even for offline ray tracers. 

A better approach is to incorporate an **acceleration structure** to quickly reject whole group of object from ray interaction testing. If you implement this correctly your time complexity is $O(m\log(n))$ were **m** is the number of pixel and **n** is the number of object in the scene.
##### Light distribution
Ray-object intersection stage gives us a point and info on the geometry. We need to figure out in addition to that how much light is hitting that scene. This is done both with geometry and radiometric distribution of light. 

So for something like a pointer light the geometry distribution of light is simply knowing the position. This isn't found in real life, to match reality we have area based lighting.

We need to worry about the amount of light power being desposited on the differential area surrounding the interaction pointer **p** We will talk about point to drill the basic concepts. This picture the general idea.

We will assume that point light has some power $\Phi$ associated with it and it radiates light equally in all direction. This means that the power-per area on a unit sphere surround the light is $\Phi/4\pi$ 

If we have a sphere that account for the power, that gets larger and further away from the point light we lose power at a rate of $1/r^2$ this is the inverse square law.

If we are interested in working out the power that's distrubited on a patch of surface, and we tilt that patch by $\theta$ from the vector from the surface of the point light, the distribution of power on the area is proportional to $\cos(\theta)$. Putting this all together we get the differential irradiance equation of

$$dE = \frac{\Phi\cos(\theta)}{4\pi r^2}$$
This is for any type of light source. You hav the two laws in the is equation the cosine falloff of light for a tilited surface and the one-over-r-squared fall off of light with distance.
##### The theory of handling more than 1 light
Illuminations is linear so you easily add more thna 1 light source. You compute the light seperately and sum them all to obtain the contribution of light.

You can randomly sample lights with a good algorithm using only some of the light sources for each shaded point in the scene because this light linearity.
##### Visibility
With ray tracing it's very easy to figure out if the light is casing a shadow from the point it's being shaded. We basically go in reverse for a given light source, we take a point on a surface and travel towards the light. These are called **shadow ray**. 

This works exactly the same as [[Ray-object Intersections]] but when we hit something we have a shadow rather than something we need to distribute power to. If there no blocking object the light constributes to the surface.
##### Light scattering at surface
With [[Ray-object Intersections]] found and the [[Light distribution]] worked out and the [[Light visibitlity]] of a surface worked out for all light. We can work out the location and incident lighthing.

We need to work out how much light is scatter back down the ray to the camera. This is the incident lighting effect. 

The incident of light arriving along the casted looks like this

![[pha01f09.svg|510]]

Were $\omega_i$ interacts with the surface of **P** and it scattered back towards the camera direction $\omega_o$. The ammount of light scattered toward the camera is given by the product of incident light energy and BRDF.
##### What is the BRDF
Each object in a scene has a material tied to it, which describes the apperabce properties at each point on a surface. The **bidirectional reflectance distribution function (BRDF)** tell us how much energy is reflected from the incoming $\hat{\omega_i}$ to an outgoing direction $\hat{\omega_o}$ 

I will refer to the **BRDF** function as $f_r(P,\hat{\omega_o}, \hat{\omega_i})$ bu conversion the direction are unit vector. This really helps with the calculations.

You can easily generalise the notion of BRDF to transmit light or scatter of light from different part of the surface. You also have the more complex **bidirectional scattering distribtion function (BSDF)** which support the scattering of light from a surface. 

The more complext function complex **bidirectional scattering surface reflectance distribution function (BSSRDF)**, this model and emit light back out that go into the surface and back out. This is necessary to correctly work with translucent materials such as milk, marble or skin.
###### Indirect light transport
The recursive nature of ray tracing allows for indirect specular reflection. We do this by reflecting a ray a surface normal at the intersection point if it's shiny. This can happen recursively, which all gets added back to the camera by following back the cameras ray eventually.

With hardware based ray tracing you can specify a limited to the amount of recusion you have in your pipeline set up. In vulkan it's under the `VkRayTracingPipelineCreationInfo::maxPipelineRayRecursionDepth`

If you do this in a compute shader, you'll need to maintain your own stack for the recursion.
###### What is stochastic progressive phonotic mapping
Since 1980 we've been able use the recusive nature of ray tracing with [[Indirect light transport for rays]] but with **stochastic progessive photon mapping (SPPM)** algorithms you are able to accurately simulate the focusing of light that passes through glass.

Here we aren't using **SPPM** and you can see the shadows are all black
![[black_shadows-1.png]]

Here we have the glass shadows accurately rendered.
![[glass_shadows-1.png]]
##### What is the light transport equation/rendering equation
The amount of light that reaches the camera froma  point is given by the sum of light emitted by an object and a light source, and the mount of light reflected. This idea is formalised in the **light transport equation**. 

This measure light with respect to radiance, a radiometric unit. The output for the equation is $L_o(P, \hat{\omega_o})$ which states that frm point **P** and direction $\omega_o$ is emitted radiance at the point of direction $L_e(P, \hat{\omega_o})$

Next we do this over all the incident radiance from all direction over a sphere from point **P** scaled by the [[What is the BRDF|BRDF]] $f_r(P,\hat{\omega_o}, \hat{\omega_i})$ and the cosine term:

$$L_o(P, \hat{\omega_o}) = L_e(P, \hat{\omega_o}) + \int\limits_{S^2} f_r(P,\hat{\omega_o}, \hat{\omega_i}) L_i(P,\hat{\omega_i}) | \cos(\theta_i) | d\hat{\omega_i}$$
You can't solve this analyticallt expect in the simplest of scene, you either must simplify the assumpts or use numerican integration technique.

Whitted's ray-tracing algorithm simplifies this integral by ingoring incoming light from most direction only in $L_i(P,\hat{\omega_i})$ for direction to light sources and directions of perfect reflections and refractions. In short this turns the integral into a small sum over over a small number of directions.

You can just randomly sample but that's very slow and you'll need to improve your algorithms to speed it up.
##### Ray propagation
When working with rays you have to be aware that you're not always passing through a vacuum. Meaning that the surface like a sphere will **NOT** always equally distribute the lights power across the surface.

You have all sorts of effects that are including Earth atmosphere causing objects that are farther away to appear less saturated.

There are two mediums you have to worry about when it comes to ray tracing. An extinguish/attenuate light either getting absorbed or scattered. We can capture this effect by computing the transmittance of $T_r$ between the origin and interfection point. When we talk about transmittance we are talking about how much light is scattered at the intersection going back to the intersection point.

So in general you have stuff like flames that emit lights and thing that scatter light through a volume. You have something called the **volume transport equation** which is used in the same way when we are working out the amount of light is reflected from a surface with the [[What is the light transport equation|light transport equation]].
##### Samples per pixel
You sample more per pixel with ray tracing to average out the result which increases accuracy. 

If you have a low amount of random samplying from an algorithm is computed. The low amount of sample the more grainy the image is because of the errors. 
##### Adding weights to rays
It's possible when you are dealing with rays that come back in the direction we care about $L_i(P,\vec{\omega_i})$ the incident spectral radiance. You can add weight to the rays which tells the image how much a individual ray contributes to an image. You can for example create a vignet, which is the camera effect of things getting darker towards the edge of the lens.
##### Basics of CPU based ray tracing
You can't analytically solve the [[What is the light transport equation|light transport equation]], so to numberically solve this integration to figure out how much light is hitting the camera, you can do something like a basic Monte Carlo, were we just randomally select a series of points and we fire rays at them from the camera, following the ray through the scene if it comes back and how it changed.

Even if you have something close to 8192 samples per pixel because we are randomly selecting points, you'll still get a tonne of noise, plus the most basic approach you wont handle perfectly specular surfaces they will just stay black. The glasses on the table should be specularily lit not black.

![[monte-carlo-basic-ray-trace-2.png]]

##### Breaking down the light transport equation
Here we are breaking the light transport equation to make it actually possible to compute. We are going to use random samplying return values for both of the [[What is the light transport equation|light transport equation]]. Here we are using the BSDF not the BRDF function.

$$L_o(P, \vec{\omega_o}) = L_e(P, \vec{\omega_o}) + \int\limits_{S^2} f(P,\vec{\omega_o}, \vec{\omega_i}) L_i(P,\vec{\omega_i}) | \cos(\theta_i) | d\vec{\omega_i}$$

It's possible with the the first part $L_e(P, \hat{\omega_o})$ emitted radiance funcion of the equation for a given ray that the ray itself might not hit anything, however the global light source that lights up everything and has no geometry to be hit, meaning that we need to add light to the scene via this ray based on the "infinite" light sources. 

The second part $\int\limits_{S^2} f(P,\vec{\omega_o}, \vec{\omega_i}) L_i(P,\vec{\omega_i}) | \cos(\theta_i) | d\vec{\omega_i}$ has a lot more going on so now we use a random direction vector $\hat{\omega'}$ . We do this because we have to workout the light value with the BSDF over the hemisphere which is extremely expensive when considering EVERY possible ray. 

So by selectin a random direction of $\hat{\omega'}$ we have equal propability over all possible directions. After that we can estimate the integral can be computed as a weight product of the BSDF which describe the light scattering properties of the material a point **P**, the $L_i(P,\hat{\omega_i})$ the incident spectral radiance and the cosine factor for the angle of the surface at point **P**
$$\int\limits_{S^2} f(P,\vec{\omega_o}, \vec{\omega_i}) L_i(P,\vec{\omega_i}) | \cos(\theta_i) | d\vec{\omega_i} \approx \frac{f(P, \vec{\omega_o}, \vec{\omega'})L_i(P, \vec{\omega'})|\cos(\theta')|}{1/4\pi} $$In other words we randomly get the direction $\hat{\omega'}$ estimating the value of the integral requires evaluating the terms in the intergrand for the drection then scale that by $4\pi$.

The intergrand is just everything after the intergal and before the differential in an intergral. 

Since you only considering 1 random direction there are error in the Monte Carlo approach, compared to the true intergal. This is the reason why we take multiple samples per pixel because on average we will get a mostly correct result. 

The **BSDF** and the **cos($\theta$)** leaving us the the $L_i(P,\vec{\omega_i})$ the incident spectral radiance to work out. To work this out we actually have to run the same process again at the origin at were the incident special radiance is, recursive repeat the process until we hit a certain depth of recursion.

To work out the **bidirectional scattering distribtion function (BSDF)** this function is working how much scattered is returning to our camera we need to get the texture and the surface properties. 

If the **BSDF** returns 0 that means that we have a non-transmissive surface. Meaning that the surface does not scatter light at all because transmissive allows light to pass through it. 

With the $|\cos(\theta')|$ and the surface normal we can compute the absolute dot function which return the absolute value of the dot product between 2 vector. That's because $\vec{a} \cdot \vec{b} = ||\vec{a}||||\vec{b}||\cos(\theta)$ that's why we care about the surface normal, and if they are normalised which they should be then $\hat{a} \cdot \hat{b} = |\cos(\theta)|$ 

Finally we need to generate a new ray from the intersection point at the position of $\vec{\omega}'$ offset enough that the float point precision isn't an issue, this is to avoid a second collision with same object again. Doing this with recurision complete the approxated function
$$\frac{f(P, \vec{\omega_o}, \vec{\omega'})L_i(P, \vec{\omega'})|\cos(\theta')|}{1/4\pi}$$
##### The short comings of a monte carlo approach to ray tracing
Follow this method [[Breaking down the light transport equation]] we have a valid ray tracing but with many short comings. 

One issues is that if the emissive surface are small (aka tiny lights) the rays will often miss them. For example if you add a point light for example, you'll get complete darkness because there is a zero probability that a ray hits that light source.

If you have **bidirectional scattering distribtion function (BSDF)** if the incident light along a direction is high the random direction vector will often miss completely, and with perfect mirrors is even more likely. 

If you improve the Monte Carlo process you can get things more accurately working however they are all going to build off the basic idea of what we did in [[Breaking down the light transport equation]]

