2026-06-12 15:06
Status: #baby 
Tags: [[graphics theory]] [[ray tracing]]
# How basic ray tracing works

You can't analytically solve the [[What is the light transport equation|light transport equation]], so to numerically solve this integration to figure out how much light is hitting the camera, you can do something like a basic Monte Carlo, were we just randomally select a series of points and we fire rays at them from the camera, following the ray through the scene if it comes back and how it changed.

Even if you have something close to 8192 samples per pixel because we are randomly selecting points, you'll still get a tonne of noise. Plus with the most basic approach you wont handle perfectly specular surfaces they will just always black. The glasses on the table should be specularily lit not black.

![[monte-carlo-basic-ray-trace-2.png]]
# References
##### Main Notes
[[Introduction to Monte Carlo]]
[[What is the light transport equation]]
#### Source Notes
[[Physically Based Rendering - Introduction]]