2026-02-11 09:25
Status: #baby 
Tags: [[vulkan]] [[logical device]]
# Modern vulkan device level features

In this modern day you'll want to set up 4 different extensions that make vulkan easier to use and update it to work with modern GPUs. 

```c++
VkPhysicalDeviceVulkan12Features vulkan12Features{};
vulkan12Features.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_2_FEATURES;
vulkan12Features.pNext = nullptr;
vulkan12Features.bufferDeviceAddress = VK_TRUE;
vulkan12Features.descriptorIndexing = VK_TRUE;
void* currentPNext = nullptr;

VkPhysicalDeviceVulkan13Features vulkan13Features{};
vulkan13Features.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_3_FEATURES;
vulkan13Features.pNext = &vulkan12Features;
vulkan13Features.dynamicRendering = VK_TRUE;
vulkan13Features.synchronization2 = VK_TRUE;
currentPNext = &vulkan13Features;
```

These features allow things to be done in an easier way or more performant if they're available.
- **.dynamicRendering** completely skips the render pass and framebuffer set up part of vulkan.
- **.synchronization2** this updates the synchronisation functions.
- **.bufferDeviceAddress** allows you to have a pointer to the `VkBuffer` in your shaders, which is one half of going bindless.
- **.descriptorIndexing** with this you treat descriptor memory as one massive array, and we can freely access any resource we want at any time, by indexing, which is the over half of going bindless.
# References
##### Main Notes
[[Device level extensions]]
[[Making GPU features usable]]
#### Source Notes
[[VulkanDev]]
