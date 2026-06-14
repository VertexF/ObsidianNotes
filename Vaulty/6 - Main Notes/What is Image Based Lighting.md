2026-06-08 10:20
Status: #baby 
Tags: [[graphics theory]] [[physically based renderer]] [[image based lighting]]
# What is Image Based Lighting

IBL or image based lighting is a collection of techniques that deal with lighting outside of point lights and ray tracing. It treats the environment like a big light source. We are basically taking our cubemap texture and using each texel as a light emitter. Which makes things look brighter and it makes everything looks like it belongs to the environment more.

So IBL is typically more accurate than even some global illumination stuff and especially ambience lighting techniques. 
# References
##### Main Notes
[[What is Physically Based Rendering]]
[[The goal with using IBL and PBR]]
#### Source Notes
[[Diffuse irradiance]]