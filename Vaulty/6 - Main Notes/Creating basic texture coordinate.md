2025-12-02 19:48
Status: #baby 
Tags: [[vulkan]] [[vulkan textures]]
# Creating basic texture coordinate

You need to have vertex attribute descriptions for the vertex buffer which is were the texcoords are gonna live.
```c++
struct Vertex 
{
    vec2s position;
    vec3s colour;
    vec2s texCoord;

    static std::array<VkVertexInputAttributeDescription, 3> getAttributeDescriptions() 
    {
        std::array<VkVertexInputAttributeDescription, 3> attributeDescriptions{};

        attributeDescriptions[0].binding = 0;
        attributeDescriptions[0].location = 0;
        attributeDescriptions[0].format = VK_FORMAT_R32G32_SFLOAT;
        attributeDescriptions[0].offset = offsetof(Vertex, position);

        attributeDescriptions[1].binding = 0;
        attributeDescriptions[1].location = 1;
        attributeDescriptions[1].format = VK_FORMAT_R32G32B32_SFLOAT;
        attributeDescriptions[1].offset = offsetof(Vertex, colour);

        attributeDescriptions[2].binding = 0;
        attributeDescriptions[2].location = 2;
        attributeDescriptions[2].format = VK_FORMAT_R32G32_SFLOAT;
        attributeDescriptions[2].offset = offsetof(Vertex, texCoord);

        return attributeDescriptions;
    }    
```

Note the format being **VK_FORMAT_R32G32_SFLOAT** matching the `vec2s`. 

You also have to have the data within your vertex buffer. Typically you wouldn't type out the texture coordinates, you would get them from the model.
```c++
const std::vector<Vertex> vertices = 
{
    {{ -0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 1.f, 0.f }},
    {{  0.5f, -0.5f }, { 1.f, 1.f, 1.f }, { 0.f, 0.f }},
    {{  0.5f,  0.5f }, { 1.f, 1.f, 1.f }, { 0.f, 1.f }},
    {{ -0.5f,  0.5f }, { 1.f, 1.f, 1.f }, { 1.f, 1.f }}
};
```

This just texture maps a sqaure. It's not that important what the data because it's just dummy data it's more that it matches with the **VK_FORMAT_R32G32_SFLOAT**. 
# References
##### Main Notes
[[Creating an combined image sampler descriptor]]
#### Source Notes
[[Texture mapping]]