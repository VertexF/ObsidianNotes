2025-09-20 15:54
Status: #baby 
Tags: [[vulkan]] [[vulkan instance]]
# Cleaning a Vulkan Instance

Cleaning up the instance should be the LAST thing you do before existing the window because literally everything is dependent on the VkInstance. 

If you've allocated the instance with a custom allocator with VMA or something else you need to deallocate the instance with the same custom allocator. This is true for all Vulkan objects.

```c++
	vkDestroyInstance(instance, nullptr);
```

# References
##### Main Notes

#### Source Notes
[[Vulkan-Tutorial]]