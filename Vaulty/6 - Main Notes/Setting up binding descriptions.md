2025-11-14 17:01
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Setting up binding descriptions

The first to do once you've set up the [[Setting up vertex buffer data]] you create vertex binding descriptions. 
```c++
static VkVertexInputBindingDescription getBindingDescriptions() 
{
    VkVertexInputBindingDescription bindingDescription{};
    bindingDescription.binding = 0;
    bindingDescription.stride = sizeof(Vertex);
    bindingDescription.inputRate = VK_VERTEX_INPUT_RATE_VERTEX;
    return bindingDescription;
}
```

The **.binding** is the index of the vertex buffer, say you have 5 vertex buffers each need there own binding so you can bind them at draw time. 

You'll find that the second argument of the **vkCmdBindVertexBuffers** which is ran at draw time request that it takes the first vertex input binding whose state is updated by the command needs to be set. Meaning that if we have a list of vertex buffer and we want to bind them all, we need to have the number equal to the lowest **.binding** from the **VkVertexInputBindingDescription** struct.

Next we need to set the stride which is the **.stride** and finally we need to set the rate on which we will send in the data this is **.inputRate**.

The **.inputRate** can be either these two values
- **VK_VERTEX_INPUT_RATE_VERTEX** Move to the next entry after each vertex.
- **VK_VERTEX_INPUT_RATE_INSTANCE** Move to the next data entry after each instance. 

This will change depending on what your vertex data is layed out like and how you're rendering.
# References
##### Main Notes
[[Setting up vertex buffer data]]
[[Setting up the vertex attributes descriptions]]
#### Source Notes
[[Vertex buffer]]
