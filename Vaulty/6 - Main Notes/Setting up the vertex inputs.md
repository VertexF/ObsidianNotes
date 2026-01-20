2025-10-25 15:55
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# Setting up the vertex inputs

#### Warning: Likely incomplete

When using a vertex buffer for vertex shaders you need to tell the vertex shader how to handle the vertex buffer. This is what this is for. This is described in 2 ways
1) **Binding** This described the the spacing between the data and if the buffer is per-vertex or per-instance.
2) **Attribute description** This describes where and how the attributes of each vertex in the buffer to the shader and what the offset is for each attribute. 

In this code example we are **NOT** going to construct the vertex buffer. To set up the vertex inputs you need to use the **VkPipelineVertexInputStateCreateInfo** struct to configure this is part of the graphics pipeline like this

```c++
VkPipelineVertexInputStateCreateInfo vertexInputState{};
vertexInputState.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
vertexInputState.vertexBindingDescriptionCount = 0;
vertexInputState.pVertexBindingDescriptions = nullptr;
vertexInputState.vertexAttributeDescriptionCount = 0;
vertexInputState.pVertexAttributeDescriptions = nullptr;
```

If you're not going to upload anything to a vertex shader this is actually complete. However, if you do want to upload data you need to set up the **.pVertexBindingDescriptions** and **.pVertexAttributeDescriptions** with arrays of structs and the count of these arrays go to the count variable in this struct.

You would do that like this
```c++
VkVertexInputBindingDescription bindingDescription = Vertex::getBindingDescriptions();
Array<VkVertexInputAttributeDescription> attributeDescriptions = Vertex::getAttributeDescriptions(renderer->stackAllocator);

VkPipelineVertexInputStateCreateInfo vertexInputState{};
vertexInputState.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
vertexInputState.vertexBindingDescriptionCount = 1;
vertexInputState.pVertexBindingDescriptions = &bindingDescription;
vertexInputState.vertexAttributeDescriptionCount = attributeDescriptions.size;
vertexInputState.pVertexAttributeDescriptions = attributeDescriptions.data;
```
# References
##### Main Notes 
[[What is the graphics pipeline]]
#### Source Notes
[[Drawing a triangle]]