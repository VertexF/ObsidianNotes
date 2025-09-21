2025-09-20 15:59
Status: #adult 
Tags: [[vulkan]] [[vulkan instance]]
# Running Vulkan on Mac

You can get the error **VK_ERROR_INCOMPLETE_DRIVER** when trying to create an instance in vulkan. With version 1.3.216 you need to have an extra extension when running on Mac called **VK_KHR_PORTABILITY_ENUMERATION_EXTENSION_NAME**
with the flag **VK_INSTANCE_CREATE_ENUMERATE_PORTABILITY_KHR** for VkInstanceCreateInfo struct.

```c++
	const char* REQUESTED_EXTENSIONS[] =
    {
        VK_KHR_SURFACE_EXTENSION_NAME,
        VK_KHR_WIN32_SURFACE_EXTENSION_NAME,
        VK_EXT_DEBUG_REPORT_EXTENSION_NAME,
        VK_KHR_PORTABILITY_ENUMERATION_EXTENSION_NAME
    };
	
    VkInstanceCreateInfo createInfo{};
	createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
	createInfo.pApplicationInfo = &applicationInfo; //The basic app set up struct
	createInfo.flags |= VK_INSTANCE_CREATE_ENUMERATE_PORTABILITY_KHR;    
    createInfo.enabledExtensionsCount = sizeof(REQUESTED_EXTENSIONS) / sizeof(REQUESTED_EXTENSIONS[0]);
    createInfo.ppEnabledExtensionsNames = REQUESTED_EXTENSIONS;
```
# References
##### Main Notes
[[First Step for setting up Vulkan]]
[[Setting up extensions]]
#### Source Notes
[[Vulkan-Tutorial]]