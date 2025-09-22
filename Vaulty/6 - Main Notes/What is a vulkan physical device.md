2025-09-22 06:40
Status: #adult  
Tags: [[vulkan]] [[vulkan physical device]]
# What is a vulkan physical device

A vulkan physical device is the direct connection from vulkan to a GPU. This is a direct connection to the GPU, but it's not used much outside creating the logical device which is an abstraction and queue families.

The physical device will be a **VkPhysicalDevice**. It's important to note you **DON'T** explicitly destroy a **VkPhysicalDevice** it's implicitly as part of the instance. 

"`VkPhysicalDevice objects cannot be explicitly destroyed. Instead, they are implicitly destroyed when the VkInstance object they are retrieved from is destroyed.`" From the spec 2.2.1 Object Lifetime - https://github.com/KhronosGroup/Vulkan-Docs/issues/68 
# References
##### Main Notes
[[Listing the GPUs in Vulkan]]
#### Source Notes
[[Vulkan-Tutorial]]