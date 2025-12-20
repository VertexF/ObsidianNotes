2025-11-16 14:43
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Allocating data to a buffer

Now we need to allocate the memory into the buffer. To begin with we need to buffer type already created to get memory requirements needed to allocate data into the buffer.

```c++
VkMemoryRequirements memoryRequirements;
vkGetBufferMemoryRequirements(device, vertexBuffer, &memoryRequirements);
```

Memory requirements for buffer are returned in this struct:
- **.size** which is the size of memory in bytes.
- **.memoryTypeBits** which is a bit mask that allows us to see if various bit types are supported. 
- **.alignment** which tells if and were the offset of buffer begins in the allocated region. This changes depending on the **VkBufferCreateInfo** **.usage** and **.flags**.

The graphics card can do a lot of different allocation, so we need to figure out which we need to use for our case.
```c++
VkMemoryAllocateInfo bufferAllocationInfo{};
bufferAllocationInfo.sType = VK_STRUCTURE_TYPE_MEMORY_ALLOCATE_INFO;
bufferAllocationInfo.allocationSize = memoryRequirements.size;

uint32_t memoryType = UINT32_MAX;
uint32_t typeFilter = memoryRequirements.memoryTypeBits;
VkMemoryPropertyFlags properties = VK_MEMORY_PROPERTY_HOST_VISIBLE_BIT | VK_MEMORY_PROPERTY_HOST_COHERENT_BIT;

VkPhysicalDeviceMemoryProperties memoryProperties;
vkGetPhysicalDeviceMemoryProperties(physicalDevice, &memoryProperties);
```

Using this function **vkGetPhysicalDeviceMemoryProperties** we get the properties for what type of heaps we can use. Within the **VkPhysicalDeviceMemoryProperties** struct you have 2 arrays.
```c++
VkMemoryType    memoryTypes[VK_MAX_MEMORY_TYPES];
VkMemoryHeap    memoryHeaps[VK_MAX_MEMORY_HEAPS];
```

There are different heaps when allocating buffers. The two types of heaps are VRAM on a graphics card or the RAM swap space on the hosts side which can be ignored and left up to the graphics driver BUT this is slower not be explict. You **HAVE** to select the memory type which is what we will do here.

```c++
uint32_t memoryType = UINT32_MAX;
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
}
```

There are 2 check here the
1) `(typeFilter & (1 << i))` **typeFilter** is a bit mask that contains 1's in binary. Each 1 tells us that a type of allocation is value. So we step through each bit checking it's with the `& (1 << i)` part.
2) Next we need to check again the appropriated memory type looping through the **memoryTypes** and comparing against the **VkMemoryPropertyFlags** we set up eariler. The reason we do an AND operation and then see if it's equal **properties** is because ANDing may leave the result as non-zero and NOT equal to the property we want.

If both things are valid now we have select the appreciate allocator for our buffer.

```c++
bufferAllocationInfo.memoryTypeIndex = memoryType;

VkDeviceMemory vertexBufferMemory;
if (vkAllocateMemory(device, &bufferAllocationInfo, nullptr, &vertexBufferMemory) != VK_SUCCESS)
{
    printf("Failed to allocate the vertex buffer object");
}

vkBindBufferMemory(device, vertexBuffer, vertexBufferMemory, 0);
```

Now we have the **memoryType** we can finish the **VkMemoryAllocateInfo** struct which is the **bufferAllocationInfo** in the code above to allocate the type memory we want.

We then bind, then sets the vertex buffer and memory to be linked together.

**vkBindBufferMemory** functions last arguments is the region in memory were the offset begins. It requires things to be divided by **.alignment** from the **VkPhysicalDeviceMemoryProperties** struct

Right now there is nothing in the memory so you'll need to fill the buffer with something before you use it. So next you need [[Copying buffer data over|copy memory]].
### **WARNING**
We are allocating memory "incorrectly" here. You're meant to allocate memory objects all together and offset into that memory because you have a hard limit of something like 4096 objects you can allocate. You can do this work yourself in vulkan or start using the VMA https://github.com/GPUOpen-LibrariesAndSDKs/VulkanMemoryAllocator 
# References
##### Main Notes
[[Copying buffer data over]]
[[Filling up a vertex buffer with CPU visibility]]
[[What is a vertex buffer]]
#### Source Notes
[[Vertex buffer]]