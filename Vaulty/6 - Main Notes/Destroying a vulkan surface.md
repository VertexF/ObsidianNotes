2025-09-26 10:17
Status: #baby 
Tags: [[vulkan]] [[vulkan surface]]
# Destroy a vulkan surface

Destroying the surface is pretty easy. You just need to do it before the instance.

```c++
vkDestroySurfaceKHR(vulkanInstance, vulkanWindowSurface, vulkanAllocationCallbacks);
```
# References
##### Main Notes
[[Creating a vulkan surface]]
#### Source Notes
[[Drawing a triangle]]