2025-11-14 16:57
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Setting up vertex buffer data

There isn't much to be said here about vertex data. This is were the vertex attributes actually are, this can contain anything that needs to be ran per-vertex. For this tutorial we are creating colour and position with a Vertex struct that contains these data types.

If you most setting up the data by look like this, normally this comes from a model.

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

It's important you know what's in the vertex data itself because this tells us what we need in the [[Shader vertex input descriptions]] the stride when [[Setting up binding descriptions]] and the individual type and stride between different vertex attributes when [[Setting up the vertex attributes descriptions]]
# References
##### Main Notes
[[Shader vertex input descriptions]]
[[Setting up binding descriptions]]
[[Setting up the vertex attributes descriptions]]
#### Source Notes
[[Vertex buffer]]