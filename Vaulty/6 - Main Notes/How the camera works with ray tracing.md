2026-06-12 08:28
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# How the camera works with ray tracing

The **eye** of a viewing volume is the place at which the viewing volume begins. When it comes to what we can see, we are only interested in the light that comes from a plane, which is what we are looking.

The camera's job is to generate a ray that travels and hits the plane in away that contributes to the light of an image. Using a origin point and direction vector this is easy to do. We might have to do more if the lenses of our camera more complex than a pin-hole camera.

Light comes from a ray that hits camera when returning to the camera has different energy levels, and just like a camera sensor we are only interested in RGB values to recreate the colour. 
# References
##### Main Notes
[[Ray-object Intersections]]
[[Differences between CPU and GPU ray tracing]]
[[What is ray tracing]]
[[Basics of ray tracing]]
#### Source Notes
[[Physically Based Rendering - Introduction]]