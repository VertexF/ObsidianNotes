2025-11-14 17:03
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]] [[vulkan shaders]]
# Setting up the vertex attributes descriptions

Here you want to describe the vertex descriptions that are in the buffer.

```c++
static std::array<VkVertexInputAttributeDescription, 2> getAttributeDescriptions() 
{
    std::array<VkVertexInputAttributeDescription, 2> attributeDescriptions{};

    attributeDescriptions[0].binding = 0;
    attributeDescriptions[0].location = 0;
    attributeDescriptions[0].format = VK_FORMAT_R32G32_SFLOAT;
    attributeDescriptions[0].offset = offsetof(Vertex, position);

    attributeDescriptions[1].binding = 0;
    attributeDescriptions[1].location = 1;
    attributeDescriptions[1].format = VK_FORMAT_R32G32B32_SFLOAT;
    attributeDescriptions[1].offset = offsetof(Vertex, colour);

    return attributeDescriptions;
}
```

So we are still at binding 0 but each location needs to be set up to match the shader. In our example, every location you see in the vertex shader is equal to the **.location**. So our `layout (location = 0) in vec2 inPosition;` is equal to `.location = 0;`

Then we have offsetof for the stride per vertex attribute and the **.format** which should be these values to match these types in the shader:

- float = VK_FORMAT_R32_SFLOAT
- vec2 = VK_FORMAT_R32G32_SFLOAT
- vec3 = VK_FORMAT_R32G32B32_SFLOAT
- vec4 = VK_FORMAT_R32G32B32A32_SFLOAT
- ivec2 = VK_FORMAT_R32G32_SINT
- uvec2 = VK_FORMAT_R32G32B32A32_SUINT
- double = VK_FORMAT_R64_SFLOAT

If you get the format wrong the **.format** this effects how the offsetof will be handled. This will be a silent failure so be careful.
# References
##### Main Notes
[[Setting up binding descriptions]]
[[Setting up vertex buffer data]]
#### Source Notes
[[Vertex buffer]]