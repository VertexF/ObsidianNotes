2025-09-24 11:32
Status: #baby 
Tags: [[vulkan]] [[logical device]]
# Cleaning up a vulkan logical device and queues

Clean up is like **VkPhysicaDevice** with **VkQueueFamily's** The **VkQueue's** you created are implicitly created with the **VkDevice** meaning when you clean up the device the queues are implicitly destroyed. You do with **VkDestroyDevice** 

```c++
vkDestroyDevice(device, vulkanAllocationCallbacks);
```

`VkQueue objects cannot be explicitly destroyed. Instead, they are implicitly destroyed when the VkDevice object they are retrieved from is destroyed.` From the spec 2.2.1 Object Lifetime - https://github.com/KhronosGroup/Vulkan-Docs/issues/68 

# References
##### Main Notes
[[Create the logical device]]
[[Cleaning up a physical device and queue families]]
#### Source Notes
[[Vulkan-Tutorial]]
