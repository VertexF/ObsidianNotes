2025-11-14 16:57
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Setting up vertex buffer data

There isn't much to be said here about vertex data. This could be just position or other stuff. For this tutorial we are creating colour and position with a Vertex struct that contains these data types.

This will eventually come from models so this isn't important really. If you most setting up the data by look like this.

```c++
struct Vertex 
{
    vec2s position;
    vec3s colour;
};

const std::vector<Vertex> vertices = 
{
    {{  0.0f, -0.5f }, { 1.f, 0.f, 0.f }},
    {{  0.5f,  0.5f }, { 0.f, 1.f, 0.f }},
    {{ -0.5f,  0.5f }, { 0.f, 0.f, 1.f }}
};
```

I will be using throughout some examples so this is purely here to link back to. However the `Vertex`class can be anything with vertices in it.
# References
##### Main Notes
#### Source Notes
[[Vertex buffer]]