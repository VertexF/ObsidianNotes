2026-06-12 08:41
Status: #baby 
Tags: [[graphics theory]] [[offline ray tracing]]
# The general approach to ray tracing

The brute force approach would be to past rays out for each object in a scene until we are done. This is extremely slow even for offline ray tracers. 

A better approach is to incorporate an **acceleration structure** to quickly reject whole group of object from ray interaction testing. If you implement this correctly your time complexity is $O(m\log(n))$ were **m** is the number of pixel and **n** is the number of object in the scene.
# References
##### Main Notes
[[Ray-object Intersections]]
[[Properties for ray tracers]]
#### Source Notes
[[Physically Based Rendering - Introduction]]
