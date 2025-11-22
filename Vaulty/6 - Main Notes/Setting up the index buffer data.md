2025-11-19 17:38
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Setting up the index buffer data

The simplest use of a index buffer would be drawing a rectangle with 2 triangles. Of course in real life you're get this data from a model file rather than this. I added this here because set up will be easier to understand.

```c++
struct Vertex 
{
    vec2s position;
    vec3s colour;
};

const std::vector<Vertex> vertices = 
{
    {{ -0.5f, -0.5f }, { 1.f, 0.f, 0.f }},
    {{  0.5f, -0.5f }, { 0.f, 1.f, 0.f }},
    {{  0.5f,  0.5f }, { 0.f, 0.f, 1.f }},
    {{ -0.5f,  0.5f }, { 1.f, 1.f, 1.f }}
};

const std::vector<uint16_t> indices = 
{
    0, 1, 2, 2, 3, 0
};
```

Index buffer will either be uint16_t or uint32_t depending on how many indices there are in the index buffer.
# References
##### Main Notes
[[What is a index buffer]]
#### Source Notes
[[Vertex buffer]]