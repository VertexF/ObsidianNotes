2025-09-20 16:11
Status: #adult 
Tags: [[vulkan]] [[vulkan instance]]
# Setting up extensions

Before you finish run vkCreateInstance with the VkInstanceCreateInfo struct. You need to decide what extensions you need to run for your Vulkan app.

Vulkan doesn't know what platform it's running on, so you plan on displaying graphics on an OS you need to set up a extension for the OS you're running on. Which would look like this.

```c++
	const char* REQUESTED_EXTENSIONS[] =
    {
        VK_KHR_SURFACE_EXTENSION_NAME,
        VK_KHR_WIN32_SURFACE_EXTENSION_NAME,
        VK_EXT_DEBUG_REPORT_EXTENSION_NAME,
        VK_EXT_DEBUG_UTILS_EXTENSION_NAME
    };
        VkInstanceCreateInfo createInfo{};
	createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
	createInfo.pApplicationInfo = &applicationInfo; //The basic app set up struct
    createInfo.enabledExtensionsCount = sizeof(REQUESTED_EXTENSIONS) / sizeof(REQUESTED_EXTENSIONS[0]);
    createInfo.ppEnabledExtensionsNames = REQUESTED_EXTENSIONS;
```

The **REQUESTED_EXTENSIONS** contains a list of const char* to different extensions you need. So **enabledExtensionsCount** just takes the size of the array and **ppEnabledExtensionsNames** takes the array.
# References
##### Main Notes
[[First Step for setting up Vulkan]]
[[Checking for supported extensions]]
[[Loading extended functions dynamically]]
[[Device level extensions]]
#### Source Notes
[[Vulkan-Tutorial]]