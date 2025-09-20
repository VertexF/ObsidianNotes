# Reference https://vulkan-tutorial.com/

#### Overview
Vulkan is a graphics API from khronos and it's a successor to OpenGL. It's cross-platform works across all the GPUs. 

Back in the day you would just send in the vertex data in a standard format, then the graphics driver takes over the everything including shading lighting. Now graphics venders have slowly updated the hardware and the graphics drivers have slowly opened to fit the programmers intent.

Tiled rendering happens on mobile GPUs and another concern with older graphics APIs is that they aren't really multithreading friend which can bottleneck stuff on the CPU side.

Vulkan unifies compute and graphics work under one system instead of only being for graphics like OpenGL.
##### Instance
The setting up the Vulkan instance should be the first thing you should be doing. This connects the application to the Vulkan library. This is truly a library you will find this is one of the part that comes the Vulkan dll vulkan-1.dll. You need to set up some details about the application before you can create the instance. VkInstance is the Vulkan instance struct.

Firstly you set up the VkApplication struct whish is actually optional but you add this because it helps with optimising the graphics driver.

```c++
void createInfo()
{
	VkApplicationInfo infoApplication{};
	infoApplication.sType = VK_STRUCTURE_TYPE_APPLICATION_INFO;
	infoApplication.pApplicationName = "Name";
	infoApplication.applicationVersion = VK_MAKE_VERSION(1, 0, 0);
	infoApplication.pEngineName = "No Engine";
	infoApplication.engineVersion = VK_MAKE_VERSION(1, 0, 0);
	infoApplication.apiVersion = VK_API_VERSION_1_0;
}
```

Many vulkan structs need to fill out the .sType for the struct name and the .pNext is used to add extensions later. It's important that you initialise the vulkan structs were possible to 0 with {};

Setting up the VkInstanceCreateInfo is not optional and is used to create the global extensions needed throughout the program. This is also were the application info is connected to via a pointer.

```c++
	VkInstanceCreateInfo createInfo{};
	createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
	createInfo.pApplicationInfo = &applicationInfo;
```

Remember Vulkan doesn't care what platform it's on. Meaning if you want to use Windows or Linux you've got to add that WSI extension to get Vulkan to work at all. 

How this works you build a C-style array and the count. The C-style array contains all the extension you want to active. All the names in the array resolve to const char* string.

```c++
	const char* REQUESTED_EXTENSIONS[] =
    {
        VK_KHR_SURFACE_EXTENSION_NAME,
        VK_KHR_WIN32_SURFACE_EXTENSION_NAME,
        VK_EXT_DEBUG_REPORT_EXTENSION_NAME
    };
    
    createInfo.enabledExtensionsCount = sizeof(REQUESTED_EXTENSIONS) / sizeof(REQUESTED_EXTENSIONS[0]);
    createInfo.ppEnabledExtensionsNames = REQUESTED_EXTENSIONS;

```

If you've done everything correct you can run the code to create instance.

```c++
	VkResult result = vkCreateInstance(&createInfo, nullptr, &instance);
```

In vulkan there is a general pattern you should know, when you use this functions the first argument is often a struct for set up, followed by the allocation callback, followed by the output variable. 

You'll find that eventually other functions have more variables arguments but this pattern is everywhere.

##### Instance set up can fail.
If you've ran the vkCreateInstance function you can get the error **VK_ERROR_INCOMPLETE_DRIVER**
This might be because you're trying to run on Mac. What you need to do with vulkan version 1.3.216 have to use the extension VK_KHR_PORTABILITY_subset. So set up should look like this.

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

If you do that with VK_KHR_PORTABILITY_ENUMERATION_EXTENSION_NAME and or the flag on whatever you have with VK_INSTANCE_CREATE_ENUMERATE_PORTABILITY_KHR vkCreateInstance should work fine.
##### Support Extension Checking.
When you run the vkCreateInstance function can get the **VK_ERROR_EXTENSION_NOT_PRESENT**
This means the extension is support and if the error happens it can close the application before anything starts running. This might be correct if the computer doesn't support the Window System Interface we need like Windows or whatever. However, you need to correctly handle the optional one.

To know what's available we need to run the vkEnumerateInstanceExtensionProperties function twice. First to get the totally number of extensions available on specified validation layer, which is the first argument follows by a nullptr third argument. If you want all available extensions then just set the first arg to nullptr. 

After you've got the totally number of extensions, you then fill up a pre-allocated array with all the extensions supported. 

```c++
        uint32_t numInstanceExtensions;
        vkEnumerateInstanceExtensionProperties(nullptr, &numInstanceExtensions, nullptr);
        VkExtensionProperties* extensions = reinterpret_cast<VkExtensionProperties*>(air_alloca(sizeof(VkExtensionProperties) * numInstanceExtensions, stackTempAllocator));
        vkEnumerateInstanceExtensionProperties(nullptr, &numInstanceExtensions, extensionsDebug);

        for (size_t i = 0; i < numInstanceExtensions; ++i)
        {
	        //Check for extensions by accessing the flat extensions[i] to see if you're thing is supported
        }
```

```c++
reinterpret_cast<VkExtensionProperties*>(air_alloca(sizeof(VkExtensionProperties) * numInstanceExtensions, stackTempAllocator));
```

Is just allocating an array with temporary piece of memory. stackTempAllocator is a stack allocator. You can make life a lot easier if you just use regular std::vector for holding the data.
##### Cleaning up
Cleaning up the instance should be the LAST thing you do before existing the window because literally everything is dependent on the VkInstance. It's important to note when you deallocating something, if that thing has been allocated with a custom allocator like VMA, you'll need to deallocate the object with the same thing. Deallocating the instances looks like this.

```c++
	vkDestroyInstance(instance, nullptr);
```
