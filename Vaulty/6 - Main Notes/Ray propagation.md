2026-06-12 09:16
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Ray propagation

When working with rays you have to be aware that you're not always passing through a vacuum. Meaning that the surface like a sphere will **NOT** always equally distribute the lights power across the surface.

You have all sorts of effects that are including Earth atmosphere causing objects that are farther away to appear less saturated.

There are two mediums you have to worry about when it comes to ray tracing. An extinguish/attenuate light either getting absorbed or scattered. We can capture this effect by computing the transmittance of $T_r$ between the origin and intersection point. When we talk about transmittance we are talking about how much light is scattered at the intersection going back to the camera point.

So in general you have stuff like flames that emit lights and thing that scatter light through a volume like mist. You have something called the **volume transport equation** which is used in the same way when we are working out the amount of light that is reflected from a surface with the [[What is the light transport equation|light transport equation]]. 
# References
##### Main Notes
[[Basics of ray tracing]]
[[How the camera works with ray tracing]]
[[Ray-object Intersections]]
[[Properties for ray tracers]]
[[The general approach to ray tracing]]
[[Light distribution]]
[[The theory of handling more than 1 light]]
[[Light visibitlity]]
[[Light scattering at a surface]]
[[Indirect light transport for rays]]
#### Source Notes
[[Physically Based Rendering - Introduction]]
