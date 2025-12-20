2025-11-21 10:38
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Making a basic createBuffer function

I'm making this note to show what's in this createBuffer function which is used everywhere. This function just creates a buffer. That's all it does. It combined both [[Setting up the a buffer]] and [[Allocating data to a buffer]] into a function.

There are draw back to this function 
1) I'm not using VMA at all here so we are limited to 4096 buffers we can create.
2) I'm not worrying about memory heap locations (this maybe solved with VMA I don't know)

```c++
static void createBuffer(VkDeviceSize size, VkBufferUsageFlags usage, VkMemoryPropertyFlags properties, VkBuffer& buffer, VkDeviceMemory& bufferMemory) 
{
    //Vertex buffer creation.
    VkBufferCreateInfo bufferInfo{};
    bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
    bufferInfo.size = size;
    bufferInfo.usage = usage;
    bufferInfo.sharingMode = VK_SHARING_MODE_EXCLUSIVE;
    
    if (vkCreateBuffer(device, &bufferInfo, nullptr, &buffer) != VK_SUCCESS)
    {
        printf("Failed to create vertex buffer.");
        assert(false);
    }

    VkMemoryRequirements memoryRequirements;
    vkGetBufferMemoryRequirements(device, buffer, &memoryRequirements);

    VkMemoryAllocateInfo bufferAllocationInfo{};
    bufferAllocationInfo.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
    bufferAllocationInfo.allocationSize = memoryRequirements.size;

    // Find memory type
    uint32_t memoryType = UINT32_MAX;
    uint32_t typeFilter = memoryRequirements.memoryTypeBits;
    VkPhysicalDeviceMemoryProperties memoryProperties;
    vkGetPhysicalDeviceMemoryProperties(physicalDevice, &memoryProperties);

    for (uint32_t i = 0; i < memoryProperties.memoryTypeCount; ++i)
    {
        if ((typeFilter & (1 << i)) && (memoryProperties.memoryTypes[i].propertyFlags & properties) == properties)
        {
            memoryType = i;
        }
    }

    if (memoryType == UINT32_MAX)
    {
        printf("Failed to find suitable memory type.");
        assert(false);
    }

    bufferAllocationInfo.memoryTypeIndex = memoryType;

    if (vkAllocateMemory(device, &bufferAllocationInfo, nullptr, &bufferMemory) != VK_SUCCESS)
    {
        printf("Failed to allocate the vertex buffer object");
        assert(false);
    }

    vkBindBufferMemory(device, buffer, bufferMemory, 0);
}
```
# References
##### Main Notes
[[Making vertex and index buffer faster with aliasing]]
[[Creating the index buffer with staging buffers]]
[[Creating the vertex buffer with staging buffers]]
[[Creating uniform buffers]]
#### Source Notes
