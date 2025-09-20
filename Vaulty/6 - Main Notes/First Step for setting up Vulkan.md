2025-09-20 16:04
Status: #baby 
Tags: [[vulkan]] [[vulkan instance]]
# First Step for setting up Vulkan

Before you start creating anything with Vulkan it's best to set up the optional struct that helps with optimisation of the graphics driver.

If this is your first time learning about how to set up Vulkan note that the .sType and the .pNext and used constantly in setting up these structs. .sType sets the type of the struct and .pNext is used to extend the struct.

```c++
	VkApplicationInfo infoApplication{};
	infoApplication.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;
	infoApplication.pApplicationName = "Name";
	infoApplication.applicationVersion = VK_MAKE_VERSION(1, 0, 0);
	infoApplication.pEngineName = "No Engine";
	infoApplication.engineVersion = VK_MAKE_VERSION(1, 0, 0);
	infoApplication.apiVersion = VK_API_VERSION_1_0;
```

You only need to worry about the **VK_API_VERSION** here as this changes what is available to you as the programmer. Newer version might not be supported on people graphics cards/devices. 
# References
##### Main Notes
[[Setting up extensions]]
[[Checking for supported extensions]]
[[Running Vulkan on Mac]]
#### Source Notes
[[Vulkan-Tutorial]]