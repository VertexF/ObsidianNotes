2026-06-12 08:23
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# Basics of ray tracing

You first need to be able to figure out were a ray interact on a given object. We need to know the properties of a given object, such as the materials and normals etc. So ray traces have the ability testing the intersection of rays with mulit object and returning the closest intersection along ray.

We also need to be able work out the distribution of light and location of lights. This includes how the ray distributes their energy throughout a scene.

So we need to be able to figure if an ray is block by something we do this by making sure that the position of the ray comes from a light, hits a object we are interested. We only return the shortest ray.

We also need to make sure that know how the light was scattered on a surface, typically this is model surfaces are parameterized so they can simulate varity of appearance.

You might need to worry about what the ray passes through such as smoke, fog and Earth atmosphore.
# References
##### Main Notes
[[What is ray tracing]]
[[How the camera works with ray tracing]]
[[Differences between CPU and GPU ray tracing]]
#### Source Notes
[[Physically Based Rendering - Introduction]]