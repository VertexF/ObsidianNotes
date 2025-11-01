2025-10-25 17:06
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# Setting up the input assembly

The input assembly is how the vulkan should handle the vertices. You have the these enum types to interpret vertices
- **VK_PRIMITIVE_TOPOLOGY_POINT_LIST** - points from the vertices
- **VK_PRIMITIVE_TOPOLOGY_LINE_LIST** - line from every 2 vertices without reuse
- **VK_PRIMITIVE_TOPOLOGY_LINE_STRIP** - Basically a big continous line has reuse
- **VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST** - triangle from every 3 vertices without any reuse
- **VK_PRIMITIVE_TOPOLOGY_TRIANGLE_STRIP** - the second and third vertex of every triangle are used as the first two vertices of the next triangle

This is pretty much the stardard set up for games.
```c++
VkPipelineInputAssemblyStateCreateInfo inputAssembly{};
inputAssembly.sType = VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO;	
inputAssembly.topology = VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST;
inputAssembly.primitiveRestartEnable = VK_FALSE;
```

Normally you have a vertex buffer and index buffer that describes vertices in sequence order. However, you can just an element buffer and you specify the indices yourself. Doing this can help with vertex optimisation reuse.

If **.primitiveRestartEnable** is set to VK_TRUE, when using any of the `_STRIP` topologies, you can specify in the buffers special breaks for when you need to break the geometry with the indices `0xFFFF` and `0xFFFFFFFF` 
# References
##### Main Notes
[[Setting up the vertex inputs]]
[[What is the graphics pipeline]]
#### Source Notes
[[Vulkan-Tutorial]]