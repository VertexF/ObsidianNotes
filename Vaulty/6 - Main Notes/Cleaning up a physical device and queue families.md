2025-09-24 11:42
Status: #adult  
Tags: [[vulkan]] [[vulkan physical device]]
# Cleaning up a physical device queue families

The physical device will be a **VkPhysicalDevice**. It's important to note you **DON'T** explicitly destroy a **VkPhysicalDevice** it's implicitly as part of the instance. The **VkQueueFamily's** also get **destroyed** with the vkDestroyInstance.

"`VkPhysicalDevice objects cannot be explicitly destroyed. Instead, they are implicitly destroyed when the VkInstance object they are retrieved from is destroyed.`" From the spec 2.2.1 Object Lifetime - https://github.com/KhronosGroup/Vulkan-Docs/issues/68 
# References
##### Main Notes
[[What is a vulkan physical device]]
[[Cleaning up a vulkan logical device and queues]]
#### Source Notes