2025-11-21 10:52
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Creating uniform buffers

Uniform buffer are a little different if you're planning to use them every frame it doesn't make sense to actually stage them because we are calculating stuff per frame on the host, you'll want make it host visible and **NOT** use staging buffers. You instead want to leave the buffer open to have data written to it every frame with **vkMapMemory**

As we have muiltple frames in flight that will be using the buffer we also need to have a buffer data for each frame. 

```c++
std::vector<VkBuffer> uniformBuffers;
std::vector<VkDeviceMemory> uniformBuffersMemory;
std::vector<void*> uniformBuffersMapped;
```

Then we create the buffers with the **createUniformBuffers** function which takes our vector grows them to size and create a host visible buffer with our limited [[Making a basic createBuffer function|createBuffer]] function

```c++
static void createUniformBuffers()
{
    VkDeviceSize bufferSize = sizeof(ModelData);

    uniformBuffers.resize(imageCount);
    uniformBuffersMemory.resize(imageCount);
    uniformBuffersMapped.resize(imageCount);

    for (uint32_t i = 0; i < imageCount; ++i)
    {
        createBuffer(bufferSize, VK_BUFFER_USAGE_UNIFORM_BUFFER_BIT, VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT, uniformBuffers[i], uniformBuffersMemory[i]);

        vkMapMemory(device, uniformBuffersMemory[i], 0, bufferSize, 0, &uniformBuffersMapped[i]);
    }
}
```

Here each buffer is mapped with **vkMapMemory** for the entire life time of the program. By not unmapping the buffer we are doing something called persistent mapping, this means we have a pointer that points to memory in the **uniformBuffersMemory** vector we can then write to this memory later on. This increase performance over re-mapping over and over every frame as mapping memory isn't free.

Like all other buffers we need to destroy them
```c++
for (uint32_t i = 0; i < maxFramesInFlight; ++i) 
{
    vkDestroyBuffer(device, uniformBuffers[i], nullptr);
    vkFreeMemory(device, uniformBuffersMemory[i], nullptr);
}

vkDestroyDescriptorSetLayout(device, descriptorSetLayout, nullptr);
```
# References
##### Main Notes
[[Updating uniform buffers]]
[[Making a basic createBuffer function]]
#### Source Notes
[[Uniform buffers]]
