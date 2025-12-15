2025-12-06 21:11
Status: #baby 
Tags: [[depth testing]] [[graphics theory]]
# What is reverse-Z perspective

When you follow tutorials online you'll find they do everything in the standard approach but the problem with with the regular depth buffer is that the precision of floating point values isn't infinite and we need to decide were the precision is going to be used for our depth buffer.

The precision of the depths is 1/z were z is the z the world space depth. This will map to 0, and 1. If you have a far plane and a near plane N and F you have this equation for the depth value d

**d = N(1/z) + F**

Since F and N are constants the d is mapped directly with 1/z

Lets map a float 32-bit value to a given depth value

![[regular_depth_z.png]]

You can see for each depth value we lose tonne of values near the vertical asymptotic. The reason here is that the 0 part of the y-axis has a lot more values when the graph line exploding to infinity but what if we reverse the y-axis? Well you get this

![[reverse_Z_depth.png]]

The depth values increase a 0 very quickly but this is where the line is levelling out to a constant value at the far plane. This massively improves the precision within the depth buffer. Espcially if you use 32-bit floats instead of 24 byte ints. This is called reverse-Z perspective!
# References
##### Main Notes
[[Setting up reverse perspective for your depth buffer]]
#### Source Notes
[[Depth buffering]]
