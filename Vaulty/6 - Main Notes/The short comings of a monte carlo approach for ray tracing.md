2026-06-12 15:12
Status: #baby 
Tags: [[graphics theory]] [[ray tracing]]
# The short comings of a monte carlo approach for ray tracing

Following this method of [[Breaking down the light transport equation]] we have valid ray tracing but with many short comings. 

One issues is that if the emissive surface are small (aka tiny lights) the rays will often miss them. For example if you add a point light for example, you'll get complete darkness because there is a zero probability that a ray hits that light source.

If you have **bidirectional scattering distribtion function (BSDF)** if the incident light along a direction is high the random direction vector will often miss it completely, and with perfect mirrors it's even more likely. 

If you improve the Monte Carlo process you can get things more accurately working however they are all going to build off the basic idea of what we did in [[Breaking down the light transport equation]]
# References
##### Main Notes
[[Introduction to Monte Carlo]]
[[How basic ray tracing works]]
[[What is the BRDF]]
[[What is the light transport equation]]
#### Source Notes
[[Physically Based Rendering - Introduction]]
