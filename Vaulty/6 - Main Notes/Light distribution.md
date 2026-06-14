2026-06-12 08:43
Status: #baby 
Tags: [[graphics theory]] [[light theory]]
# Light distribution

When it comes [[Ray-object Intersections|Ray-object intersection]] for ray tracing it gives us a point and the geometric information. We need to figure out in addition to that how much light is hitting that scene. This is done both with geometry and radiometric distribution of light. 

So for something like a point light the geometric distribution of light is simply knowing the position. This isn't found in real life, to match reality we have to use area based lighting.

We need to worry about the amount of light power being desposited on the differential area surrounding the interaction pointer **P** We will talk about point lights to drill the basic concepts. This picture gives the general idea.
![[pointLight-1.png|656]]

We will assume that point light has some power $\Phi$ associated with it and it radiates light equally in all direction. This means that the power-per area on a unit sphere surround the light is $\Phi/4\pi$ which is based on the area of a sphere.

If we have a sphere that account for the power, that gets larger and further away from the point light we lose power at a rate of $1/r^2$ this is the inverse square law.

If we are interested in working out the power that's distributed on a patch of surface, and we tilt that patch by $\theta$ from the vector from the surface of the point light, the distribution of power on the area is proportional to $\cos(\theta)$. Putting this all together we get the differential irradiance equation of

$$dE = \frac{\Phi\cos(\theta)}{4\pi r^2}$$
This is for any type of light source. You have the two laws in the is equation the cosine falloff of light for a tilited surface and the one-over-r-squared fall off of light with distance.

# References
##### Main Notes
[[Ray-object Intersections]]
[[The theory of handling more than 1 light]]
#### Source Notes
[[Physically Based Rendering - Introduction]]