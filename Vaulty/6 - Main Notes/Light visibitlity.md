2026-06-12 08:57
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Light visibitlity

With ray tracing it's very easy to figure out if the light is casing a shadow from the point it's being shaded. We basically go in reverse for a given light source, we take a point on a surface and travel towards the light. These are called **shadow rays**. 

This works exactly the same as [[Ray-object Intersections]] but when we hit something we have a shadow rather than something we need to distribute power to. If there no blocking object the light constributes to the surface.
# References
##### Main Notes
[[The general approach to ray tracing]]
[[Ray-object Intersections]]
[[Properties for ray tracers]]
[[How the camera works with ray tracing]]
#### Source Notes
[[Physically Based Rendering - Introduction]]