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

#### Validation Layers

Vulkan was designed to have as little to do with with driver as possible, which has some draw back. Simple, complex, easy to miss mistakes aren't handled at all. If you get something wrong with Vulkan it will continue with undefined behaviour, or simply just crash. The worst case is it works on your graphics card but doesn't work on other people. So in comes the validation layer. 

Vulkan is layered meaning that different layers can sit between functions calls and proxy out function calls while doing extra work underneath. These are all completely optional.

The validation layer will do thing such as checking values for arguments. Tracing, profiling and replying different calls. Checking thread safety by keeping track of the creation of that thread and what it's up to. It check the creation and deletion order of objects is create, making sure resource leaks aren't happening. Logging all the errors and none errors to standard out or error out streams.

These layers can be stacked on each other or completely removed for a release build. LunarG provided validation layers as part of the SDK. 

There were once instance level and device level. Device validation was meant to check things against GPU specifically. Now device validation have been completely deprecated and only global instance level validation layers are now used.

It is recommend that you turn on device level validation anyway for combability reasons with older GPU this can be done easily enough when you create the logical device.

##### How to use validation layers

When you are turning on validation layers you only really need to turn on the **"VK_LAYER_KHRONOS_validation"** as a const char*.

It's important to note that the validation layers need to be turn on at VkInstance creation and at logical device creation. You likely don't want this is you're compiling for release so a good system is to turn on and off setting the validation layers based on NDEBUG or a "#define" of your own, like I have done.

```c++
	const char* REQUESTED_LAYERS[] =
    {
#if defined(VULKAN_DEBUG_REPORT)
        "VK_LAYER_KHRONOS_validation"
#else
        ""
#endif
    };

        VkInstanceCreateInfo createInfo = {};
        createInfo.sType = VK_STRUCTURE_TYPE_INSTANCE_CREATE_INFO;
        createInfo.pNext = nullptr;
        createInfo.flags = 0;
        createInfo.pApplicationInfo = &applicationInfo;
#if defined(VULKAN_DEBUG_REPORT)
        createInfo.enabledLayerCount = ArraySize(REQUESTED_LAYERS);
        createInfo.ppEnabledLayerNames = REQUESTED_LAYERS;
#else
        createInfo.enabledLayerCount = 0;
        createInfo.ppEnabledLayerNames = nullptr;
#endif //VULKAN_DEBUG_REPORT
        createInfo.enabledExtensionCount = ArraySize(REQUESTED_EXTENSIONS);
        createInfo.ppEnabledExtensionNames = REQUESTED_EXTENSIONS;
```

Please pay attention to the fact that these layers are **enabledLayerCount** and **ppEnabledLayerNames** these are layers NOT extensions so they set up as a different part of the struct. Another thing to note that is check if the validation layer is there is pointless as any vulkan developer should have vulkan SDK installed and if you're running this on someone else computer should turn off the validation layers.

#### How to set up message callbacks
The validation layer will print stuff to the standard output stream by default, but adding a function callback is a good idea as you can add more stuff and format the output to fit your needs. Filter messages out is often something you will want to do so you don't see literally everything. 

Okay so what do you need to do? First thing is get the function callback extension and turn it on. **VK_EXT_DEBUG_REPORT_EXTENSION_NAME** it might be worth checking if this is available, but again if you have the SDK installed you will also have this extension installed. Note this is exactly like setting any other extension. 

Now we have to write the right function to match the callback. This should be a static function that contains both **VKAPI_ATTR** and **VKAPI_CALL** the function should look something like this.

```c++
	static VKAPI_ATTRI VkBool32 VKAPI_CALL debugCallback
	(VkKDebugUtilsMessageSeverityFlagBitsEXT messageSeverity,
	VkDebugUtilsMessageTypeFlagsEXT messageType,
	cosnt VkDebugUtilsMessengerCallbackDataEXT* callbackData,
	void* userData)
	{
		std::cerr << "Validation error :" << callbackData->pMessage << std::endl;
		
		return VK_FALSE;
	}
```

Now the function can be named anything but this is the structure it has to follow. The first argument **VkKDebugUtilsMessageSeverityFlagBitsEXT** and this can be any in four states. 
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT` Diagnostic info not meant for you.
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_INFO_BIT_EXT`General normal info of things working.
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT` Like a bug, but not fatal.
- `VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT` A fatal bug that needs to be solved.

You can simply do a comparison on these values in the debug function callback to only print stuff you're interested. I personally cause `__debugbreak();` function called with both warning and error.

The next argument is can tell use what the message is, there are three things this can be.
- `VK_DEBUG_UTILS_MESSAGE_TYPE_GENERAL_BIT_EXT` This is for any old event unrelated to performance or the specification. 
- `VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT` This is for when you've done something wrong against the specification.
- `VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT`This is for when there is a potential performance issue.

The third argument **VkDebugUtilsMessengerCallbackDataEXT** is the most useful as this contains the error message itself. This struct contains
- `pMessage` The debug message in a null-terminated string (aka const char*)
- `pObjects` This is a flat array of objects that are related to the message.
- `objectCount` This contains the number of objects in the array.

Finally the final pointer contains a pointer to data the user created on setup of this function.

You should always return VK_FALSE when running this function and abort when the message validation error has been triggered. If you return true, you will get a **VK_ERROR_VALIDATION_FAILED_EXT** which is meant for testing the validation layer themselves which is completely useless to you.

To actually set this up correctly you need to set up and tear down the function callback. You can have more than one if you want to be part of the debug messenger.

```c++
        VkDebugUtilsMessengerCreateInfoEXT creationInfo = {};
        creationInfo.sType =                                                                               VK_STRUCTURE_TYPE_DEBUG_UTILS_MESSENGER_CREATE_INFO_EXT;
        creationInfo.pNext = nullptr;
        creationInfo.pfnUserCallback = debugUtilsCallback;
        creationInfo.messageSeverity =                                                                     VK_DEBUG_UTILS_MESSAGE_SEVERITY_ERROR_BIT_EXT |
                        VK_DEBUG_UTILS_MESSAGE_SEVERITY_WARNING_BIT_EXT |
                        VK_DEBUG_UTILS_MESSAGE_SEVERITY_VERBOSE_BIT_EXT;
         creationInfo.messageType =                                                                        VK_DEBUG_UTILS_MESSAGE_TYPE_PERFORMANCE_BIT_EXT |                                  VK_DEBUG_UTILS_MESSAGE_TYPE_VALIDATION_BIT_EXT;
         createInfo.pUserData = nullptr;
```

This is what I've set up in the past. **messageSeverity** will tell vulkan what message severity you're interested in. **messageType** tells vulkan what type of messages you want to know about under the different levels of severity. **pfnUserCallback** is just a handle to the function pointer you should've already set up. **pUserData** can take in a pointer to a struct or class what can provide extra info. So if you have a 1 big struct with everything in, you can pass structs pointer in here and access it later when an error happens.

To actually run this function you need to load a function before you can run it as it's part of the debug utils extensions and not part of core vulkan. This is similar to what you need to do for OpenGL with glew that loads all the functions.

```c++
            PFN_vkCreateDebugUtilsMessengerEXT vkCreateDebugUtilsMessengerEXT =(PFN_vkCreateDebugUtilsMessengerEXT)vkGetInstanceProcAddr(vulkanInstance, "vkCreateDebugUtilsMessengerEXT");
```

To do this you need to the vkGetInstanceProcAddr function, the instance and the correct string that gives you the function you're interested in running, in this case **"vkCreateDebugUtilsMessengerEXT"** Once you've done that you can then simply use the variable **vkCreateDebugUtilsMessengerEXT** in the above code as a function call. If this fails it's likely due to the fact you haven't set up the **VK_EXT_DEBUG_UTILS_EXTENSION_NAME** extension up.

After that you just run the function with all the details needed

```c++
vkCreateDebugUtilsMessengerEXT(vulkanInstance, &debugMessenegerCreateInfo, vulkanAllocationCallbacks, &vulkanDebugUtilsMessenger);
```

Every argument here is pretty standard apart from the last one. You have a vulkan instance first, second you have the debug messenger struct you have already set up, then the vulkan allocator. Finally you get a **VkDebugUtilsMessengerEXT** output variable which seems to keep track of the life span of the vulkan message callback. This is used for decollate and destroy the function.

If you've ran this function with no errors the validation layer should now report back to your custom function. 

Finally you can clean up the debug extension callback with a similar idea. The destruction of the debug util messenger also needs a function to be loaded, just like it's creation, but it's very similar as creation.

```c++
        auto vkDestroyDebugUtilsMessengerEXT = (PFN_vkDestroyDebugUtilsMessengerEXT)vkGetInstanceProcAddr(vulkanInstance, "vkDestroyDebugUtilsMessengerEXT");
        vkDestroyDebugUtilsMessengerEXT(vulkanInstance, vulkanDebugUtilsMessenger, vulkanAllocationCallbacks);
```

Okay so we have the **vkDestroyDebugUtilsMessengerEXT** with 3 arguments the vulkan instance, the vulkanDebugUtilsMessenger is a variable of the **VkDebugUtilsMessengerEXT** which tells vulkan to deload all the debug function callbacks. Finally we have the allocator again, remember with the allocator you allocate and deallocate in the same way. 

When setting up this you will notice that VkCreateInstance and VkDestroyInstance can't be debugged because you need the instance is created before the callback is created. However Khronos designed something for this.

If you create the **VkDebugUtilsMessengerCreateInfoEXT** first and then pass this into the **VkInstanceCreateInfo** .pNext variable with the enabling of the validation layer and debug util extension and you'll get validation layer feedback on the creation and deletion on the VkInstance.

```c++
        createInfo.enabledLayerCount = ArraySize(REQUESTED_LAYERS);
        createInfo.ppEnabledLayerNames = REQUESTED_LAYERS;
        createInfo.enabledExtensionCount = ArraySize(REQUESTED_EXTENSIONS);
        createInfo.ppEnabledExtensionNames = REQUESTED_EXTENSIONS;
		//Creation of the VkDebugUtilsMessengerCreateInfoEXT in the function call.
        const VkDebugUtilsMessengerCreateInfoEXT debugCreateInfo = createDebugUtilsMessengerInfo();
        createInfo.pNext = &debugCreateInfo;
```

#### Configuration
You can configure the validation layers in different ways beyond the VkDebugUtilsMessengerCreateInfoEXT flags contains in that struct. If you go your vulkan SDK and look for the **Config** directory, you will find a file called **vk_layer_settings.txt** that file should explain how to configure things.

You'll need to do some project configuration to customise your validation layer usage. You will need to copy your layer settings to debug and/or release directories in your project once you've changed things. If you don't do that you'll be running with the debug set of validation layers, which is totally chill.

#### Setting up Physical Device
After we have initialised the vulkan instances we need to select a physical device we will do all the GPU work on. You can select more than 1 to do different things in parallel but this complicated the things, but could be worth it.

To select the physical device, you need to use **vkEnumeratePhysicalDevices** once to get the amount of physical and second to get the actual list of physical device. If you have listed the extensions before it's the same process.

```c++
        uint32_t numPhysicalDevice;
        result = vkEnumeratePhysicalDevices(vulkanInstance, &numPhysicalDevice, nullptr);
        check(result);

        VkPhysicalDevice* gpus = reinterpret_cast<VkPhysicalDevice*>(air_alloca(sizeof(VkPhysicalDevice) * numPhysicalDevice, stackTempAllocator));
        result = vkEnumeratePhysicalDevices(vulkanInstance, &numPhysicalDevice, gpus);
        check(result);
```
With the list of physical device in a flat array you can now select which one you want from a for loop.