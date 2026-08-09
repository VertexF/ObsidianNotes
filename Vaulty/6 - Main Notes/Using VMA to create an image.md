2026-07-25 10:59
Status: #baby 
Tags: [[vulkan]] [[vulkan image views]] 
# Using VMA to create an image

Generally you would want to set up things like for images.
```c++
VmaAllocationCreateInfo memoryInfo{};
memoryInfo.flags = VMA_ALLOCATION_CREATE_DEDICATED_MEMORY_BIT;
memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;

check(vmaCreateImage(gpu.VMAAllocator, &imageInfo, &memoryInfo, &texture->vkImage, &texture->vmaAllocation, nullptr));
```

Using **VMA_MEMORY_USAGE_AUTO** will allow VMA to select the best piece of memory for the process. Also using the **VMA_ALLOCATION_CREATE_DEDICATED_MEMORY_BIT** allows for VMA to create a new allocator for big resources, this is good for large image attachments.
# References
##### Main Notes
[[Creating a texture with KTX file]]
[[Hidden issues with VMA host visible buffers]]
[[CPU updating coherent buffers]]
[[Using VMA to create a GPU buffer]]
[[Setting up VMA]]
#### Source Notes
[[How to vulkan]]