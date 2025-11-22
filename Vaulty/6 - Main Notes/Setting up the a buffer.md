2025-11-16 14:40
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Setting up the a buffer

In this example I will use a vertex buffer set up but any type of buffer goes through the same thing.

```c++
VkBufferCreateInfo bufferInfo{};
bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
bufferInfo.size = sizeof(vertices[0]) * vertices.size();
bufferInfo.usage = VK_BUFFER_USAGE_VERTEX_BUFFER_BIT;
bufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;
```

We have the **.size** which is the size of the buffer that's going to be used. **.usage** is the usage of the buffer, you can OR more usage types together if you want. Finally we have **.sharingMode** which like the swapchain images, you can specify if you're doing to use them with different queue families or not. We aren't going to use this any where other than the graphics queue.

```c++
VkBuffer vertexBuffer;
if (vkCreateBuffer(device, &bufferInfo, nullptr, &vertexBuffer) != VK_SUCCESS) 
{
    printf("Failed to create vertex buffer.");
}
```

Then you create the buffer. Deleting is very similar

```c++
vkDestroyBuffer(device, vertexBuffer, nullptr);
```
# References
##### Main Notes
[[What is a vertex buffer]]
#### Source Notes
[[Vertex buffer]]
