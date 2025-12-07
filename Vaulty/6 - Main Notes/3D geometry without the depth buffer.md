2025-12-06 17:01
Status: #baby 
Tags: [[vulkan]] [[depth testing]]
# 3D geometry without the depth buffer

If you do everything needed to have a piece of 3D geometry added to your scene without a depth buffer, you'll have every piece of 3D geometry overlapping other piece of geometry in ways you wouldn't expect.

```c++
const std::vector<Vertex> vertices = 
{
    {{ -0.5f, -0.5f, 0.f }, { 1.f, 1.f, 1.f }, { 1.f, 0.f }},
    {{  0.5f, -0.5f, 0.f }, { 1.f, 1.f, 1.f }, { 0.f, 0.f }},
    {{  0.5f,  0.5f, 0.f }, { 1.f, 1.f, 1.f }, { 0.f, 1.f }},
    {{ -0.5f,  0.5f, 0.f }, { 1.f, 1.f, 1.f }, { 1.f, 1.f }},

    {{ -0.5f, -0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 1.f, 0.f }},
    {{  0.5f, -0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 0.f, 0.f }},
    {{  0.5f,  0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 0.f, 1.f }},
    {{ -0.5f,  0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 1.f, 1.f }}
};
```

As you can see with the vertices at -0.5f for the second piece of geometry but if you render this you get this result

![[overlapping_geometry.png]]

This happens because the second part of the geometry in the index buffer come after the first.

```c++
const std::vector<uint16_t> indices = 
{
    0, 1, 2, 2, 3, 0, //first
    4, 5, 6, 6, 7, 4  //second
};
```

You have two ways of solving the ordering of fragments so the depth is correct.
1) Sorting your indices front to back based on depth
2) Using a depth testing with a depth buffer.

Sorting geometry back to front with the index buffer is actually how you do order-dependent transparency, as the independent version is hard to do. What's most common and practical is sorting fragments with a depth buffer. 

A depth buffer is another attachment that will be part of the graphics pipeline. This holds the depth values for every position, like the colour attachment holds colour values for every position. Everytime the rasteriser produces a fragment, it will do a depth test to check if the new fragment is closer or further away from the previous one. If it **ISN'T** closer to the screen then the fragment gets discarded. Each time the fragment **IS** closer to the screen the fragment writes its depth value to the depth buffer.

You can modify the depth values in the fragment shader the same way you can modify the colour values.
# References
##### Main Notes
[[Creating the depth image and view]]
[[Setting up vertex buffer data]]
[[Setting up the index buffer data]]
#### Source Notes
[[Depth buffering]]