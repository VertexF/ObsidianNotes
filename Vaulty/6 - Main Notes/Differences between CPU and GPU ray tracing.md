2026-06-12 09:40
Status: #baby 
Tags: [[graphics theory]] [[ray tracing]]
# Differences between CPU and GPU ray tracing

The algorithms you writing in CPU and GPU are the same the biggest difference between the two is setup, pipelining. With non-RT stuff is more similar to the CPU raytracer, you just use a compute shader to do it, but queries are way less flexible and a lot slower.
# References
##### Main Notes
[[What is ray tracing]]
[[Basics of ray tracing]]
[[How the camera works with ray tracing]]
#### Source Notes
