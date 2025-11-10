# Reference https://vulkan-tutorial.com/Drawing_a_triangle/Setup/Base_code

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

#### Checking hardware combability for our Vulkan application.
We will want to check the Vulkan support version, GPU type and properties so we can get the right GPU we want. This is done with the **vkGetPhysicalDeviceProperties** function.

```c++
VkPhysicalDeviceProperties properties;
vkGetPhysicalDeviceProperties(hardwareDevice, &properties);
```

It's important to note though if you want to support optional feature like texture compression and doubles and halfs you need to query with the **vkGetPhysicalDeviceFeatures** and **VkPhysicalDeviceFeatures**. 

```c++
VkPhysicalDeviceFeatures features;
vkGetPhysicalDeviceFeatures(hardwareDevice, &features);
```

There additional things that can be queried like GPU memory but will will talk about the later in this source note.

**vkGetPhysicalDeviceProperties** and **vkGetPhysicalDeviceFeatures** are just structs that contain a list of data members that are true or false depending on the queried function. An example let see if device is discrete and can support geometry shaders.

```c++
bool isDeviceSuitable(VkPhysicalDevice device)
{
	VkPhysicalDeviceProperties properties;
	VkPhysicalDeviceFeatures features;
	vkGetPhysicalDeviceProperties(hardwareDevice, &properties);
	vkGetPhysicalDeviceFeatures(hardwareDevice, &features);
	
	return properties.deviceType == VK_PHYSICAL_DEVICE_TYPE_DISCRETE_GPU && features.geometryShader;
}
```

It's very common for people to have more than one GPU in there computer. So what you can do is loop through all the hardware and pick the best one to operate on if you're planning on using just 1. You can go through the features and properties you care about, give them a score based on what properties and features the GPUs have and pick the one with the highest score to use. Just make sure you have a fallback if you don't find one with the properties want.
#### Queue Families
Vulkan works with commands and these commands are submitted to a queue. The different type of queues that we can use for different commands come from queue families. So a queue that comes from a queue family can only use a subset of commands. For example, if I have a queue that handles memory transfer commands, then this queue can't handle graphics or computer commands.

To be able to set up the queues to submit commands, we need to check what queues families are supported by the graphics card, as it's possible to have a GPU that only support memory transfer or compute. So this wouldn't be good for rendering.

To list all the queue families from a physical device we do the same old get function thing **vkGetPhysicalDeviceQueueFamilyProperties** we run this once to get the count and again to the the actual list.

```c++
uint32_t queueFamilyCount = 0;
vkGetPhysicalDeviceQueueFamilyProperties(vulkanPhysicalDevice, &queueFamilyCount, nullptr);

VkQueueFamilyProperties* queueFamilies = reinterpret_cast<VkQueueFamilyProperties*>(air_alloca(sizeof(VkQueueFamilyProperties) * queueFamilyCount, stackTempAllocator));

vkGetPhysicalDeviceQueueFamilyProperties(vulkanPhysicalDevice, &queueFamilyCount, queueFamilies);
```

After you've extracted thing the struct **VkQueueFamilyProperties** contains the properties of the queue families. Just like with getting the physical device we need to loop through them all and selected on we want. For rendering you at least need a queue family type that has **VK_QUEUE_GRAPHICS_BIT**. 

Here's some code on how to do this, but if you don't know how use this as example and code the stuff yourself. You need practice if you don't know how.

```c++
uint32_t mainQueueFamilyIndex = UINT32_MAX;
uint32_t computeQueueFamilyIndex = UINT32_MAX;
uint32_t computeQueueIndex = UINT32_MAX;
for (uint32_t familyIndex = 0; familyIndex < queueFamilyCount; ++familyIndex)
{
	//Get the queue family we are working on.
	VkQueueFamilyProperties queueFamily = queueFamilies[familyIndex];
	//Reject that queue family if he has nothing in it.
    if (queueFamily.queueCount == 0)
    {
	    continue;
    }
    
    //Main queue is the one with both graphics and compute
    if ((queueFamily.queueFlags & (VK_QUEUE_GRAPHICS_BIT | VK_QUEUE_COMPUTE_BIT)) == (VK_QUEUE_GRAPHICS_BIT | VK_QUEUE_COMPUTE_BIT))
    {
	    //The index of main and compute are the family next.
        mainQueueFamilyIndex = familyIndex;

        if (queueFamily.queueCount > 1)
        {
            computeQueueFamilyIndex = familyIndex;
            computeQueueIndex = 1;
        }
        continue;
    }
}
```
#### Logical Device
The logical device is the interface between the physic device and Vulkan commands, you can have more than one logical device if you need it. 

When creating the logical device you should the **VkPhysicalDevice** and the queue family indices you've gotten from GPU. After you've done that you need to set up the **VkDeviceQueueCreateInfo** this with that struct you can create the queues with the given queue family index. Then you can create the queues with the **VkDevice** creation.
#### Setting up the queues
So you simply say how many queues you want to create from a given queue family and that's how you create it.

```c++
VkDeviceQueueCreateInfo queueCreateInfo{};
queueCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
//This is our main index we found from going over the queue families eariler.
queueCreateInfo.queueFamilyIndex = mainQueueFamilyIndex;
//You can create more than 1 queue if you need it here.
queueCreateInfo.queueCount = 1;
```

You often don't want to create more than 1 queue from a given queue family. This because the drivers only really let you create a few and having more than 1 causes more graphic driver overhead. If you want to handle a lot of commands can create the command buffers on multiple threads and submit all at once on the main thread.

You also need the priorities these queues with a value from 1.f to 0.f. These values helps with scheduling of the command buffer execution. You have to do this even if you only have one queue.

```c++
queueCreateInfo.pQueuePriorities = 1.f;
```
#### Activating device features
When you're trying to use features that you have access to when you query the physical device with **vkGetPhysicalDeviceFeatures** function you need to create structs that turn them on for the logical device. 
#### Creating the local device
If you've created the **VkDeviceQueueCreateInfo** and the **VkPhysicalDeviceFeatures2** you're finally ready to create the local device with the creation struct **VkDeviceCreateInfo**.

```c++
VkDeviceCreateInfo createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
createInfo.pQueueCreateInfos = &queueCreateInfo;
createInfo.queueCreateInfoCount = 1;
createInfo.pEnabledFeatures = &deviceFeatures;
```

The queue is the number of queues that belong your **VkDeviceQueueCreateInfo** for example in my engine I have a static array of 3, **BUT** it's not always 3 because the GPU might only have 1, 2 or 3 separate queue families for me to use for a given type. 

The rest is a little tricky conceptually. You need to now turn on device specific extensions in the struct. It's important to not that global based extensions which you have been turned on earlier are different from device level extension. It's possible that we are on a Windows machine, with the instance level Windows extension turned but the GPU we've selected doesn't have the ability to display any output and only uses compute. This is why we need to turn on device level extensions, which can't be known at instance level creation.

The most important device level extension we need for rendering is the **VK_KHR_swapchain** this allows us to render things to rendering things to the screen.

Layer are also strange remember when I said talked about [[Instance and device level validation]] well this is were would would turn on device level validation which will most likely be ignored. 

This is snippet of what I've done to turn on device level extensions. 

```c++
        Array<const char*> deviceExtensions;
        deviceExtensions.init(tempAllocator, 2);
        deviceExtensions.push(VK_KHR_SWAPCHAIN_EXTENSION_NAME);
        deviceExtensions.push(VK_KHR_SHADER_DRAW_PARAMETERS_EXTENSION_NAME);
        
        VkDeviceCreateInfo deviceCreateInfo = {};
        deviceCreateInfo.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
        deviceCreateInfo.queueCreateInfoCount = queueCount;
        deviceCreateInfo.pQueueCreateInfos = queueInfo;
        deviceCreateInfo.enabledExtensionCount = deviceExtensions.size;
        deviceCreateInfo.ppEnabledExtensionNames = deviceExtensions.data;
        deviceCreateInfo.pNext = &physicalDeviceFeature2;
```

After you've done that you can finally create the logical device with **vkCreateDevice** Just make sure to check if it worked or not.

```c++
result = vkCreateDevice(vulkanPhysicalDevice, &deviceCreateInfo, vulkanAllocationCallbacks, &vulkanDevice);

check(result);
```

Arguments are 
- 1) The VkPhysicalDevice 
- 2) The VkDeviceCreateInfo 
- 3) The VMA/allocator callbacks.
- 4) The VkDevice

Clean up is like **VkPhysicaDevice** with **VkQueueFamily's** The **VkQueue's** you created are implicitly created with the **VkDevice** meaning when you clean up the device the queues are implicitly destroyed. You do with **VkDestroyDevice** 

```c++
vkDestroyDevice(device, vulkanAllocationCallbacks);
```
#### Getting the queue handles
As the Queue are automatically created with the creation of the logical device, you don't have access to them straight away. You do need to create **VkQueue** for the different queues you have created with the logical device. 

So to load the VkQueue into memory to be used you would so something like
```c++
vkGetDeviceQueue(vulkanDevice, computeQueueFamilyIndex, computeQueueIndex, &vulkanComputeQueue);
```

The arguments in order are:
- 1) The VkDevice
- 2) The family queue index value. 
- 3) The queue index
- 4) The VkQueue
It's important to make sure that the queue index you got from queue family index are being used together in this argument or you'll likely get a validation error when trying to create the VkQueue. 
#### Window surface
Since Vulkan is platform agnostic it doesn't directly touch the windowing system. To get Vulkan to be able to present result to the screen we need to turn on the Window System Integration (WSI) extension. There are at least two and one of them is the **VK_KHR_surface**. It allows use to use the **VkSurfaceKHR** object which is an abstract object that allows use to present rendered images to. 

The window surface actually needs to be created right after instance creation, because it will influence what physical device you want to use. The window surface is actually completely optional in Vulkan, if you want to do offscreen rendering you just don't create any surface to deal with the images. Unlike OpenGL which requires you to hack an invisible window.
#### Window surface creation.
We are going to create the **VkSurfaceKHR**. This swapchain is platform agnostic meaning that we need to use a platform specific to get thing working with our swapchain and since we're on Windows we need to use the **VK_KHR_win32_surface** which is found in the **VK_KHR_WIN32_SURFACE_EXTENSION_NAME** name.

In my code I've been using this extension list for agers

```c++
    const char* REQUESTED_EXTENSIONS[] =
    {
        VK_KHR_SURFACE_EXTENSION_NAME,
        VK_KHR_WIN32_SURFACE_EXTENSION_NAME,
#if defined(VULKAN_DEBUG_REPORT)
        VK_EXT_DEBUG_REPORT_EXTENSION_NAME,
        VK_EXT_DEBUG_UTILS_EXTENSION_NAME
#endif//VULKAN_DEBUG_REPORT
    };
```

You can create the surface yourself or with SDL or GLFW. If you create it yourself you'll have to do it for all the platforms what you'll end up supporting. To do this for windows you need to get the window instance and the window handle. 

```c++
//get this from SDL/GLFW or the Win32 API
HINSTANCE windowInstance;
HWND window;
VkWin32SurfaceCreateInfoKHR createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_WIN32_SURFACE_CREATE_INFO_KHR;
createInfo.hwnd = window;
createInfo.hinstance = windowInstance;

VkResult result = vkCreateWin32SurfaceKHR(instance, &createInfo, vulkanAllocatorCallback, &surface);
```

If you do everything right you should end up with a surface. However you can just use GLFW or SDL to do the work for you.

```c++
glfwCreateWindowSurface(instance, window, nullptr, &surface);

SDL_Window* window = reinterpret_cast<SDL_Window*>(creation.window);
if (SDL_Vulkan_CreateSurface(window, vulkanInstance, &vulkanWindowSurface) == SDL_FALSE)
{
    aprint("Failed to create Vulkan surface.\n");
}
SDLWindow = window;
```

Destroying the surface is pretty straight forward, just do it before the instance destruction.

```c++
vkDestroySurfaceKHR(vulkanInstance, vulkanWindowSurface, vulkanAllocationCallbacks);
```
#### Seeing if presentation is supported.
This is a queue family thing. When you're looking for if the has draw command support for graphics you also need to look for presentation support. It's possible for a graphics queue family to **NOT** support presentation. 

To do this you need to check with the function **vkGetPhysicalDeviceSurfaceSupportKHR** this takes in the physical device, the surface you've created and check against the queue family index you've found to see if it has presentation support.

Here's what I've done. In this code assume that we've already done the work to extract the queue families from the hardware.
```c++
VkBool32 surfaceSupport = VK_FALSE;
for (uint32_t familyIndex = 0; familyIndex < queueFamilyCount; ++familyIndex)
{
	VkQueueFamilyProperties queueFamily = queueFamilies[familyIndex];
	if (queueFamily.queueCount > 0 && queueFamily.queueFlags &                            (VK_QUEUE_GRAPHICS_BIT | VK_QUEUE_COMPUTE_BIT))
        {
            check(vkGetPhysicalDeviceSurfaceSupportKHR(physicalDevice, familyIndex, vulkanWindowSurface, &surfaceSupport));

                // If we have some how found a queue family 
                // that support graphics and compute commands 
                // but not presention we need to skip that one.
                if (surfaceSupport)
                {
                    vulkanMainQueueFamily = familyIndex;
                    break;
                }
            }
        }
```

In my code we are looking for a queue family index that has graphic, queue and presentation support. The variable **surfaceSupport** returns true or false depending if presentation queues are supported.

Why am I looking for a queue family that has graphics, compute and presentation and setting that as the "main" family queue index. Simple, performance. It's faster for graphics commands to happen on the same queue were handle images.

In my code if I don't find a queue family index that has presentation, graphics and compute I close the program out right. This might not be the best idea, but it make sure I get the best performance.
#### Creating the presentation queue
When creating the logical device it's important to remember that the VKQueue's are created with the VkDevice. We now have the queue family index that has presentation support now we need to create the VkQueue that support presentation.

Okay so this is going to be very dependent on that fact you've already worked out that the presentation family queue **IS PART** of the graphics queue family. If this is true then the main family index that contains presentation, graphics and queue will have been created with the VkDevice already. 

```c++
VkDeviceQueueCreateInfo& mainQueue = queueInfo[queueCount++];
mainQueue.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
mainQueue.queueFamilyIndex = mainQueueFamilyIndex;
mainQueue.queueCount = computeQueueFamilyIndex == mainQueueFamilyIndex ? 2 : 1;
mainQueue.pQueuePriorities = queuePriority;
```

If you have **NOT** don't do this you'll need create a separate presentation VkQueue that's not part of the graphics queue because you don't know if the GPU has a queue family that supports both.

```c++
float queuePriority = 1.f;
std::vector<VkDeviceQueueCreateInfo> queueFamilyList;
for (uint32_t queueFamily : uniqueQueueFamilies)
{
	VkDeviceQueueCreateInfo& queueInfo = queueInfo[queueCount++];
	queueInfo.sType = VK_STRUCTURE_TYPE_DEVICE_QUEUE_CREATE_INFO;
	queueInfo.queueFamilyIndex = queueFamily;
	queueInfo.queueCount = 1;
	queueInfo.pQueuePriorities = queuePriority;
	queueFamilyList.push_back(queueInfo);
}
```

Then you can add either version of the **VkDeviceQueueCreateInfo** to the VkDevice creation. Remember if you're doing graphics queues and presentation queues you'll need to retrieve the presentation queue.
#### Swapchain
Vulkan doesn't have a default framebuffer is. The buffer handling for double/triple buffer we will need to handled by you. This is what a swapchain is for. All that a vulkan swapchain is a series of images waiting to be rendered in a queue. How it works is that we acquire and image, render on it and return it to the swapchain.

This is a device level extension you might want to check is supported, if it's not supported then it's okay to close the application right away. You do this in the standard way.

```c++
	uint32_t extensionCount = 0;
	vkEnumerateDeviceExtensionProperties(physicalDevice, nullptr, &extensionCount, nullptr);
	std::vector<VkExtensionProperties> availableExtensions(extensionCount);
	vkEnumerateDeviceExtensionProperties(physicalDevice, nullptr, &extensionCount, availableExtensions.data());
```

Now you have to check if what you have is in the list of availableExtensions. You can do this anyway you want for example: 

```c++
std::vector<const char*> deviceExtensions;
for (uint32_t i = 0; i < availableExtensions.size(); ++i) 
{
	if (strcmp(availableExtensions[i].extensionName, VK_KHR_SWAPCHAIN_EXTENSION_NAME))
	{
		deviceExtensions.push_back(VK_KHR_SWAPCHAIN_EXTENSION_NAME);
		continue;
	}
}
```

#### Checking swapchain support
You're not done you need to make sure that the swapchain is compatible with our window surface. We do this by querying the details.
You need to check:
- Basice surface capabilities
	- min/max number of images the swapchain has.
	- height/width and of the swapchain images.
- Surface formats
	- pixel format
	- Colour space
- Available presentation modes

To get all these detials you'll need to fill in the structs **VKSurfaceCapabilitiesKHR** **VkSurfaceFormatKHR** and **VkPresentModeKHR**. 

```c++
	VkSurfaceCapabilitiesKHR surfaceCapabilities;
	vkGetPhysicalDeviceSurfaceCapabilitiesKHR(physicalDevice, surface, &surfaceCapabilities);
```

The VkSurfaceCapabilitiesKHR has the basic capabilities which are the min/max swapchain images for example. You need to query with the surface with a given surface and physical device. 

Now we need to query the surface format with this very fimilar pattern. So this will be the pixel format and the colour space that's supported.

```c++
	uint32_t formatCount = 0;
	vkGetPhysicalDeviceSurfaceFormatsKHR(physicalDevice, surface, &formatCount, nullptr);
	std::vector<VkSurfaceFormatKHR> surfaceFormats(formatCount);
	if (formatCount != 0) 
	{
		vkGetPhysicalDeviceSurfaceFormatsKHR(physicalDevice, surface, &formatCount, surfaceFormats.data());
	}

```

#### Chosing the right swapchain format
So after you have all the information now you need to create the swapchain and pick the right one for your currently set up. To pick the best swapchain mode we need to consider these three factors.
- Prensentation mode. How are we going to wait on the swapchain for images.
- Colour format which is the colour depth, so SRGB or SBGR 
- Swapchain extends so the width and heigh of the images.
##### Surface format
You'll need to check the formats of all the **VkSurfaceFormatKHR** to make sure you get a surface format you want for your swapchain. The default and first choose should be the **VK_FORMAT_B8G8R8A8_SRGB** this is the mostly widely supported colour space out there and it's non-linear allowing more accurate colours when your blending. Any **...A8_SRGB** should be the default for any texture too.

```c++
	VkSurfaceFormatKHR swapchainSurface{};
	for (const auto& availableSurface : surfaceFormats)
	{
		if (availableSurface.format == VK_FORMAT_B8G8R8A8_SRGB) 
		{
			swapchainSurface = availableSurface;
		}
	}
	swapchainSurface = surfaceFormats[0];
```

Note at the bottom we just select the first surface we find if there is no **VK_FORMAT_B8G8R8A8_SRGB** which is fine typically. 
##### Presentation mode
This is tided to presentation of the images when it comes to submitting drawn frames in vulkan. So for now we are only going to give a light overview of the different types of presentation modes and **NOT** how it works for now.

There are two things you have to understand about how things are rendered before you can understand the presentention mode. 

One the screen rasters from top the bottom based on the monitors Hz. So most standard monitors are at 60Hz which means it will take roughly 16.66ms to scan from top to bottom.

Two we are drawing to an image in vulkan, this has to me presented to the monitor so it can raster the image. 

- VK_PRESENT_MODE_IMMEDIATE_KHR - This very simply just takes an image that has been drawn to with the draw commands and presents it to the monitor as soon as it's down. This can result in screen tearing.
- VK_PRESENT_MODE_FIFO_KHR - This uses the queue inside the swapchain. If an image has been rendered to it gets inserted into the back of the queue. Then we waits until the monitor has rastered the image and then top pop an image off the queue to raster. If the swapchain queue is full the application has to wait. This avoid any chance of v-tearing and is often called v-sync because we are vertically synced with the monitor.
- VK_PRESENT_MODE_RELAXED_KHR - Similar to the FIFO but lets say the vertical blank was finished, the queue is empty at the last of vertical blank. Then we just submits it, this can end up with v-tearing
- VK_PRESENT_MODE_MAILBOX_KHR - This is pretty simpy if the application is fast enough that the swapchain vertical blank is taking too long we just replace the current frame in the queue.

Only VK_PRESENT_MODE_FIFO_KHR is the only that's supported. That being said MAILBOX is a good one to go for if it's available.

This is how you might want to select the presentation mode
```c++
	uint32_t presentationModeCount = 0;
	vkGetPhysicalDeviceSurfacePresentModesKHR(physicalDevice, surface, &presentationModeCount, nullptr);
	std::vector<VkPresentModeKHR> presentationModes(presentationModeCount);
	vkGetPhysicalDeviceSurfacePresentModesKHR(physicalDevice, surface, &presentationModeCount, presentationModes.data());
	
	VkPresentModeKHR presentationMode = VK_PRESENT_MODE_MAX_ENUM_KHR;
	for (uint32_t i = 0; i < presentationModes.size(); ++i)
	{
		if (presentationModes[i] == VK_PRESENT_MODE_MAILBOX_KHR)
		{
			presentationMode = VK_PRESENT_MODE_MAILBOX_KHR;
			break;
		}

		presentationMode = VK_PRESENT_MODE_FIFO_KHR;
	}
```

##### Swapchain extents
Okay so we need to get the swapchain extents to match the current window. This is a little weird because of the fact that some window managers allow different values than the size of the window. So we are looking for is the window width and height of **minImageExtents** and **maxImageExtent**. 

Why do we need to do this? For high DPI settings, which changes how many pixels are in a resolution. Like Apple's Retian display has more pixels for the given resolution meaning if you just read the pixel values you're swapchain extents will be off. 

So to get this right you need to use the https://wiki.libsdl.org/SDL2/SDL_Vulkan_GetDrawableSize SDL_Vulkan_GetDrawableSize or the GLFW glfwGetFramebufferSize. But return the true drawable area in pixels account for high DPI screens. **NOTE:** In SDL3 this function doesn't exists.
##### Creating the swapchain
After you've found the correct properties for a given swapchain, you need to select how many images you want to deal with. You can do this by querying **VkSurfaceCapabilitiesKHR** minimum image count + 1. 

```c++
	uint32_t imageCount = surfaceCapabilities.minImageCount + 1;
```

The reason you don't want just the minimum is that the graphics driver might be doing we have to wait around for, so it's better to have 1 more than the minimum so we don't have to wait.

We also need to make sure we didn't exceed the maximum number of images used. You can't always say that you can use three or four, as that might be bigger than the maximum allowed. If there is 0 then we are being told there is **NO** maximum image value.

```c++
	if (surfaceCapabilities.maxImageCount > 0 && imageCount > surfaceCapabilities.maxImageCount)
	{
		imageCount = surfaceCapabilities.maxImageCount;
	}
```

After you've done that you start filling up the swapchain creation struct.

Starting with the basics we have

```c++
	VkSwapchainCreateInfoKHR createSwapchain{};
	createSwapchain.sType = VK_STRUCTURE_TYPE_SWAPCHAIN_CREATE_INFO_KHR;
	createSwapchain.surface = surface;
	createSwapchain.minImageCount = imageCount;
	createSwapchain.imageFormat = swapchainSurface.format;
	createSwapchain.imageColorSpace = swapchainSurface.colorSpace;
	createSwapchain.imageExtent = swapchainExtent;
	createSwapchain.imageArrayLayers = 1;
	createSwapchain.imageUsage = VK_IMAGE_USAGE_COLOR_ATTACHMENT_BIT;
```

- **imageArrayLayers** should be 1 unless you're doing something with stereoscopic 3D application.
- **imageUsage** is about what you're going to be writing to the swapchain, if you're writing to an image first to then do post-processing you might want to use **VK_IMAGE_USAGE_TRANSFER_DST_BIT** instead of the colour bit version which means we are writing directly to the swapchain images.

Now we have to worry about the two different modes for dealing with queues. `VK_SHARING_MODE_EXCLUSIVE` and `VK_SHARING_MODE_CONCURRENT` for out 
**.imageSharingMode** of the struct.

These are to do with queue families and if you have more than 1 for the presentation and graphics. If the presentation queue and graphics queue are the same, then you use `MODE_EXCLUSIVE`, if they are different then you use `MODE_CONCURRENT`.

By far the most common is when the presentation queue and the graphics queue are on the same queue family. This is more performant so this is what you want. The code below shows you the difference but it's dummy code. We are assuming that **presentationQueueFamily** and **graphicsQueueFamily** have been acquired earlier in the code base.

```c++
	uint32_t presentationQueueFamily = 0;
	uint32_t graphicsQueueFamily = 0;
	std::vector<uint32_t> queueFamilyProps;
	queueFamilyProps.push_back(presentationQueueFamily);
	queueFamilyProps.push_back(graphicsQueueFamily);
	if (queueFamilyProps.size() == 1 || queueFamilyProps[0] != queueFamilyProps[1])
	{
		createSwapchain.imageSharingMode = VK_SHARING_MODE_CONCURRENT;
		createSwapchain.queueFamilyIndexCount = 2;
		createSwapchain.pQueueFamilyIndices = queueFamilyProps.data();
	}
	else 
	{
		createSwapchain.imageSharingMode = VK_SHARING_MODE_EXCLUSIVE;
		createSwapchain.queueFamilyIndexCount = 0;
		createSwapchain.pQueueFamilyIndices = nullptr;
	}
```

You also need to specify if the images you're dealing with require rotation or not. If you don't want rotation you just query the surface supported capalities with **.currentTransform**

```c++
createSwapchain.preTransform = surfaceCapabilities.currentTransform;
```

Next you need to say if you want images alpha values to be blended with other windows in the window system. You almost always want to ingore this with **VK_COMPOSITE_ALPHA_OPAQUE_BIT_KHR**

Next week need to set the presentation mode for our swapchain. We can also say that for our swapchain if we want to keep the pixels that are behind a window. So if you have firefox partly in the way of your game do you want to clip those pixels or not. You might want to keep them if you need them for graphical effects but if you don't need the pixels clip them.

```c++
	createSwapchain.presentMode = presentationMode;
	createSwapchain.clipped = VK_TRUE;
```

You also have the **.oldSwapchain** which needs to be feed with the old swapchain when you need to reconstruct the swapchain after a window resize event. If you don't care about resize windows then set it to null.

After that you can create the swapchain with the struct you just made. Destroying follows the same pattern as other destruction functions too. The first arguments of both functions is the logical device.

```c++
	VkSwapchainKHR swapchain;
	vkCreateSwapchainKHR(device, &createSwapchain, nullptr, &swapchain);
	
	vkDestroySwapchainKHR(device, swapchain, nullptr);
```
##### Getting the swapchain images
Now we have created the swapchain we now need a container to store the swapchain **VKImage's** These images will be created from the swapchain itself and like VkQueue's and VkPhysicalDevice they doesn't need cleaning up and will be destroy the swapchain is.

We have specified the minimum number of swapchain images that can exist with the **VkSurfaceCapabilitiesKHR** struct with the code

```c++
surfaceCapabilities.minImageCount + 1;
```

The swapchain has that information and now it can create a larger number of VKImages. It's the same pattern you've seen before.

```c++
	uint32_t imageCount = 0;
	vkGetSwapchainImagesKHR(device, swapchain, &imageCount, nullptr);
	std::vector<VkImage> swapchainImages(imageCount);
	vkGetSwapchainImagesKHR(device, swapchain, &imageCount, swapchainImages.data());
```

If you don't have access to the **VkFormat** from the **VkSurfaceFormatKHR** struct or the **VkExtent2D** from the **VkSurfaceCapabilitiesKHR** struct you'll need both values stored for later.
##### Image views
To be able to use VkImage's including those in the swapchain you need to create VkImageView's objects. This object has information about the VkImage and how to use it. For example, it might tell you that a VkImage is a 2D depth texture that has mipmap, this would might be used for shadows or something similar.

So for the swapchain you'll need to create VkImageView's for every VkImage so you can use them as a colour target. These image views but for a swapchain images that's simply meant, but remember you can do a lot with VkImageView's. 

So for the swapchain images you need to create a **VkImageViewCreateInfo** struct for each VkImage that the swapchain has.

You would want to start out with something like this were the **.image** is equal to a swapchain image.
```c++
std::vector<VkImageView> swapchainImageViews(swapchainImages.size());
for(uint32_t i = 0; i < swapchainImages.size(); ++i)
{
	VkImageViewCreateInfo createImageViewInfo{};
	createImageViewInfo.sType = VK_STRUCTURE_TYPE_IMAGE_VIEW_CREATE_INFO;
	createImageViewInfo.image = swapchainImages[i];
}
```

Next we'll want to specify how the data should be interpreted with the **.viewType** and **.format** we get the **VK_IMAGE_VIEW_TYPE_1D|2D|3D|CUBE** for different types of data, we want 2D for a swapchain image. We also need to set it format in the **.format** and we can simply get this from our swapchain struct. 

```c++
	createImageViewInfo.viewType = VK_IMAGE_VIEW_TYPE_2D;
	createImageViewInfo.format = swapchain.format;
```

Next we want to set the components for the colour which allows us to swizzle the colour channels around. This means we can rearrange the colour channels around, set them to monochromic or even set them from a map from 1 to 0. In the swapchain context you'll want them as default. 

```c++
	createImageViewInfo.components.r = VK_COMPONENT_SWIZZLE_IDENTITY;
	createImageViewInfo.components.g = VK_COMPONENT_SWIZZLE_IDENTITY;
	createImageViewInfo.components.b = VK_COMPONENT_SWIZZLE_IDENTITY;
	createImageViewInfo.components.a = VK_COMPONENT_SWIZZLE_IDENTITY;
```

Next we need to set the **.subresourceRange** this explains what the purpose of the image is and how much of the VkImage are we doing to access. So this is were the mip levels are set. For the context of a swapchain image you'll want this set up

```c++
	createImageViewInfo.subresourceRange.aspectMask = VK_IMAGE_ASPECT_COLOR_BIT;
	createImageViewInfo.subresourceRange.baseMipLevel = 0;
	createImageViewInfo.subresourceRange.levelCount = 1;
	createImageViewInfo.subresourceRange.baseArrayLayer = 0;
	createImageViewInfo.subresourceRange.layerCount = 1;
```

Just a word of warning about **.levelCount** and **.layerCount** they can't be 0 and it's a very common mistake to type out layerCount or levelCount twice and leave the other one as 0. So be careful.

You then can simply run the **vkCreateImageView** function with the struct and fill up the array.
```c++
	if (vkCreateImageView(device, &createImageViewInfo, nullptr, &swapchainImageViews[i]) != VK_SUCCESS)
	{
		printf("Failed to create image view at index: %d", i);
		assert(false);
	}
```

Unlike VkImage's which are destroyed with the logical device, you have to clean up the image view's yourself.

```c++
	for (VkImageView imageView : swapchainImageViews) 
	{
		vkDestroyImageView(device, imageView, nullptr);
	}
```

Khronos SDK has a **vender-independent** compiler. The compiler checks your code is correct and when it's compiled you can **ship that code** with your game. If you use the glslc.exe compiler it fits closer with the paramter formats with GCC and Clang, not only that you get `#includes` for free. These are both with the vulkan SDK.
#### Creating the shader modules
This is going to part of the graphics pipeline as a **VkShaderModule**. You need to create with the good old creation struct pattern. 

Assuming you've aligned the char* pointer to a uint32_t* by [[Reading in compiled shaders]] correctly you should okay. I'm only going to show the vertex shader but any shader has the same idea.

```c++
VkShaderModule vertexShader;

VkShaderModuleCreateInfo vertexCreateInfo{};
vertexCreateInfo.sType = VK_STRUCTURE_TYPE_SHADER_MODULE_CREATE_INFO;
vertexCreateInfo.codeSize = sizeOfVertexCode;
vertexCreateInfo.pCode = reinterpret_cast<const uint32_t*>(binVertCode);

if (vkCreateShaderModule(device, &vertexCreateInfo, nullptr, &vertexShader) != VK_SUCCESS) 
{
	printf("Failed to create the vertex shader module");
}
```

Once we have created the **VkShaderModule** we don't need to keep the binary shader code around. So don't forget to free that

```c++
free((void*)binVertCode);
binVertCode = nullptr;
```

The **VkShaderModule** is a thin wrapper that is going to be uploaded to the GPU to be compiled into GPU machine code. This means that it's part of the graphics pipeline meaning we can have **destroy** the **VkShaderModules** at the end of the creation of the graphics pipeline.

```c++
//... Above this comment is all the code needed to create a graphics pipeline.

vkDestroyShaderModule(device, vertexShaderModule, nullptr);
```
##### Shader stage creation
To complete and add the shader modules to the graphics pipeline you'll need to fill the **VkPipelineShaderStageCreateInfo**. To be able to do this you'll need to have first completed the [[Creation of the shader module]] because you'll be using them here. I will do this for the vertex shader but it's very similar with other shaders.

```c++
VkPipelineShaderStageCreateInfo vertShaderModuleInfo{};
vertShaderModuleInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_SHADER_STAGE_CREATE_INFO;
vertShaderModuleInfo.stage = VK_SHADER_STAGE_VERTEX_BIT;
vertShaderModuleInfo.module = vertexShaderModule;
vertShaderModuleInfo.pName = "main";
```

So we have 3 things here:
1) The **.stage** which takes an enum for a given shader type.
2) The **.module** this takes the coorisponding shader module
3) The **.name** which takes the name of the entry point of the shader, which is pretty much always main.
The **.name** is a little interesting. It's possible to have muiltiple shaders combined in a **VkShaderModule**. If this happens we need to tell the struct what entry point depends on what shader.

**.pSpecializationInfo** is interesting because this is a value that can be used to help split the shader up into parts, as this can be used in the shader a constants value that can effectively if-def out parts of shader at compile time which is faster to do at render time. If you don't any constant values then you just leave this null.

To actually use these you need to load the **VkPipelineShaderStageCreateInfo** into a flat array to be used in the graphics pipeline. 

```c++
	VkPipelineShaderStageCreateInfo  shaderStages[] = { vertShaderModuleInfo, fragShaderModuleInfo };
```
##### Graphics Pipeline
In older APIs the graphics pipeline there are default values for the graphics pipeline but in vulkan we specify all of them. 
##### Dynamic state
Not all states of the graphics pipeline in vulkan are static you have the
- Viewport size and scissor size
- Line width
- Blend constants
All of which are things that can be changed at draw time. If you want to use these features at draw time you need to tell vulkan with the **VkPipelineDynamicStateCreateInfo**. It's very common to block out the viewport and the scissor state from the static graphics pipeline, as it allows to resizable windows.

```c++
std::vector<VkDynamicState> dynamicStates = 
{
	VK_DYNAMIC_STATE_VIEWPORT,
	VK_DYNAMIC_STATE_SCISSOR
};

VkPipelineDynamicStateCreateInfo dynamicState{};
dynamicState.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
dynamicState.dynamicStateCount = uint32_t(dynamicStates.size());
dynamicState.pDynamicStates = dynamicStates.data();
```

If you do this though you'll be required at draw time to specify the viewport state and scissor state.
##### Vertex Input
This is all to do with the vertex buffer and how this should be interpreted at the vertex shader stage of the pipeline. There are 2 ways this is described
1) **Bindings** This is the spacing between the data. This tells us if the vertex data needs to be per-vertex or per-instance for instanced rendering.
2) **Attribute descriptions** This is concerned with where and how we pass the attributes of the vertex buffer to the shader and what the offset is for each attribute.

Note since this from the Vulkan-Tutorial it actually skips over how specify a vertex buffer for later. However we still have to specify the **VkPipelineVertexInputStateCreateInfo** and add it to the graphice pipeline even if we don't plan on uploading any data to the shaders.

```c++
VkPipelineVertexInputStateCreateInfo vertexInputState{};
vertexInputState.sType = VK_STRUCTURE_TYPE_PIPELINE_VERTEX_INPUT_STATE_CREATE_INFO;
vertexInputState.vertexBindingDescriptionCount = 0;
vertexInputState.pVertexBindingDescriptions = nullptr;
vertexInputState.vertexAttributeDescriptionCount = 0;
vertexInputState.pVertexAttributeDescriptions = nullptr;
```

**.pVertexBindingDescriptions** and **.pVertexAttributeDescriptions** are both array of structs that describe the binding and the attribute descriptions and the others data members is just the count of these arrays.

NOTE: vertex attributes and bindings get a lot more complex if you require the faster bindless set up.
##### Input assembly
This is involved in specifying the topology and if the primitive restarted. The primitive can be:
- **VK_PRIMITIVE_TOPOLOGY_POINT_LIST** - points from the vertices
- **VK_PRIMITIVE_TOPOLOGY_LINE_LIST** - line from every 2 vertices without reuse
- **VK_PRIMITIVE_TOPOLOGY_LINE_STRIP** - Basically a big continous line has reuse
- **VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST** - triangle from every 3 vertices without any reuse
- **VK_PRIMITIVE_TOPOLOGY_TRIANGLE_STRIP** - the second and third vertex of every triangle are used as the first two vertices of the next triangle

```c++
VkPipelineInputAssemblyStateCreateInfo inputAssembly{};
inputAssembly.sType = VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO;	
inputAssembly.topology = VK_PRIMITIVE_TOPOLOGY_TRIANGLE_LIST;
inputAssembly.primitiveRestartEnable = VK_FALSE;
```

Normally vertices are loaded by vertex buffer and index buffer in sequential order. However, you can element buffer you can specify the indices yourself. This allows you to optimise reuse of vertices. If **primitiveRestartEnable** to be true then it's possible to use `_STRIP` topologies and break then up with special index breaks `0xFFFF`or `0xFFFFFFFF` in the buffer.
##### Viewports and scissors
A viewport basically decides what part of the framebuffer is outputted for rendering. Unless said otherwise just do from 0, 0 to swapchain's min and max values.

```c++
	VkViewport viewport{};
	viewport.x = 0.f;
	viewport.y = 0.f;
	viewport.width = float(swapchainExtent.width);
	viewport.height = float(swapchainExtent.height);
	viewport.minDepth = 0.f;
	viewport.maxDepth = 1.f;
```

Remember that swapchain VkImage's can have a different size to the window. So we use the extents over the size of the application window size. 

Maximum and minimum depth are for the swapchain. If you aren't doing anything special then setting **minDepth** 0.f and **maxDepth** to 1.f is a good idea unless your doing reverse perspective.

The viewport defines the transformation from the image to the framebuffer but the scissor rectangles define what's actually stored. Any pixel outside of the scissor will be discarded by the rasteriser. 

![[viewports_scissors-2294964527.png]]

As you can see here the scissor rectangle chops the image, while the viewport scales it. If you want the scissor to stretch the entire viewport you just do this.

```c++
	VkRect2D scissor{};
	scissor.offset = { 0, 0 };
	scissor.extent = swapchainExtent;
```

It's important to note that the viewport and the scissor most of the time should be set up a run time. You need to the dynamic state set up to have this feature enabled and it's important to note too that this cost 0 performance to do at draw time.

If you want to do that you just need to set up the **VkPipelineViewportStateCreateInfo** and set the viewport count and scissor count to see like is.

```c++
VkPipelineViewportStateCreateInfo viewportCreateInfo{};
viewportCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_VIEWPORT_STATE_CREATE_INFO;
viewportCreateInfo.viewportCount = 1;
//viewportCreateInfo.pViewports = &viewports;
viewportCreateInfo.scissorCount = 1;
//viewportCreateInfo.pScissors = &scissors;
```

If you need a static viewport you'll want to use set them here were I've commented out the code. In the command buffers you can actually set things up so you have a different number of viewports if you want. If you do want to have more than 1 viewport and scissor you'll need to turn on a device level extension.
##### Rasteriser
The rasteriser takes geometry 3D or 2D and turns them into fragments (pixel custers) that get written to by the framgment shader. It is performs depth testing, face culling, and the scissor test. It can be configured to just do the wireframe or full polygons. You configure this with the **VkPipelineRasterizationStateCreateInfo** struct.

```c++
VkPipelineRasterizationStateCreateInfo rasteriser{};
rasteriser.sType = VK_STRUCTURE_TYPE_PIPELINE_RASTERIZATION_STATE_CREATE_INFO;
rasteriser.depthClampEnable = VK_FALSE;
```

If you set the **.depthClampEnable** to VK_TRUE you'll clamp the fragments that are beyond the near and far planes rather than discarding them. This is useful to have with shadow mapping but requires a GPU feature to be turned on to use.

```c++
rasteriser.rasterizerDiscardEnable = VK_FALSE;
```

If this is set to VK_TRUE all the geometry gets completely thrown out, this rendering nothing. This will be useful if you want to profile how fast your vertex/mesh shaders are.

```c++
rasteriser.polygonMode = VK_POLYGON_MODE_FILL;
```

Next we have the polygon mode. This can be set to 3 settings and one exclusive to Nvidia cards which I will skip and requires enableding a GPU feature.
- VK_POLYGONE_MODE_FILL - This is the normal fille the entire polygon.
- VK_POLYGONE_MODE_LINE - This will only save the lines between each of the vertices.
- VK_POLYGONE_MODE_POINT - This will only render the vertices as points.

```c++
rasteriser.lineWidth = 1.f;
```

This sets the line width the rasteriser will output. Anything more than 1.f requires the GPU feature `wideLines` to be enabled. Note the different GPUs support different line widths.

```c++
rasteriser.cullMode = VK_CULL_MODE_BACK_BIT;
rasteriser.frontFace = VK_FRONT_FACE_CLOCKWISE;
```

The **cullMode** is set up to cull the back facing polygones or the front facing polygones. The **.frontFace** tells vulkan which way the polygons are facing based on it's winding. 

```c++
rasteriser.depthBiasEnable = VK_FALSE;
rasteriser.depthBiasConstantFactor = 0.f;
rasteriser.depthBiasClamp = 0.f;
rasteriser.depthBiasSlopeFactor = 0.f;
```

This four lines of code set things up so there is a bias for the depth values. This can be useful for shadow mapping but if you're not doing that you can just leave them all to 0.
##### Multisampling
Multisampling is a way you can reduce anti-aliasing effects. It takes the work of the fragment shader that rasters to the same pixel and samples it mutliple times and does some kind of averaging to get a blend of the pixel colours around a given pixel. This is only really meant for the edges which is were aliasing effects happen. To use this feature you need to turn on GPU feature. To set this up you use the **VkPipelineMultisampleStateCreateInfo** and for this example we'll show what a multisampling set up looks like for no multisampling.

```c++
VkPipelineMultisampleStateCreateInfo multisamplingState{};
multisamplingState.sType = VK_STRUCTURE_TYPE_PIPELINE_MULTISAMPLE_STATE_CREATE_INFO;
multisamplingState.sampleShadingEnable = VK_FALSE;
multisamplingState.rasterizationSamples = VK_SAMPLE_COUNT_1_BIT;
multisamplingState.minSampleShading = 1.f; //optional
multisamplingState.pSampleMask = nullptr; //optional
multisamplingState.alphaToCoverageEnable = VK_FALSE; //optional
multisamplingState.alphaToOneEnable = VK_FALSE; //optional
```

##### Depth testing and stencil testing
Note at this point in the tutorial it is skiped for later. You do need this for anything 3D including 2D rendering with a layered 2D rendering system.
##### Colour blending
After the fragment buffer has returned a colour we need to know what to do with the colours that have already have been generated. This is know as the colour blending and there are two ways to do it
1) Mix the old and new colours to make the final colour.
2) Combine the old and new values using bitwise operations.

There are two structs you have to set up for colour blending. 
1) **VkPipelineColorBlendAttachmentState** is configuring things per framebuffer attachment.
2) **VkPipelineColorBlendStateCreateInfo** is only interested in teh global blending settings.

This is what **VkPipelineColourBlendAttachmentState** construction looks like for 1 framebuffer, with the default blending you would want for most cases.

```c++
VkPipelineColorBlendAttachmentState colourBlendAttachment{};
colourBlendAttachment.colorWriteMask = VK_COLOR_COMPONENT_R_BIT | VK_COLOR_COMPONENT_G_BIT | VK_COLOR_COMPONENT_B_BIT | VK_COLOR_COMPONENT_A_BIT;

colourBlendAttachment.blendEnable = VK_FALSE;
colourBlendAttachment.srcColorBlendFactor = VK_BLEND_FACTOR_ONE;
colourBlendAttachment.dstColorBlendFactor = VK_BLEND_FACTOR_ZERO;
colourBlendAttachment.colorBlendOp = VK_BLEND_OP_ADD;
colourBlendAttachment.srcAlphaBlendFactor = VK_BLEND_FACTOR_ONE;
colourBlendAttachment.dstAlphaBlendFactor = VK_BLEND_FACTOR_ZERO;
colourBlendAttachment.alphaBlendOp = VK_BLEND_OP_ADD;
```

If you look at the above struct with it's variables like **.blendEnable** this is basically what is happening in the pseudocode.
```c++
if(blendEnable)
{
	finalColour.rgb = (srcColourBlendFactor * newColour.rgb) <colorBlendOp> (dstColorBlendFactor * oldColour.rgb);
	
	finalColour.a = (srcAlphaBlendFactor * newColour.rgb) <alphaBlendOp> (dstAlphaBlendFactor * oldColour.a);
}
else
{
	finalColour = newColour;
}

finalColour = finalColour & colourWriteMask;
```

What the final the final line here does is it's determinting the colour channel that gets outputted.

What you might actually want to do with this is alpha blending. Will look like this.

```c++
colourBlendAttachment.blendEnable = VK_TRUE;
colourBlendAttachment.srcColorBlendFactor = VK_BLEND_FACTOR_SRC_ALPHA;
colourBlendAttachment.dstColorBlendFactor = VK_BLEND_FACTOR_ONE_MINUS_SRC_ALPHA;
colourBlendAttachment.colorBlendOp = VK_BLEND_OP_ADD;
colourBlendAttachment.srcAlphaBlendFactor = VK_BLEND_FACTOR_ONE;
colourBlendAttachment.dstAlphaBlendFactor = VK_BLEND_FACTOR_ZERO;
colourBlendAttachment.alphaBlendOp = VK_BLEND_OP_ADD;
```

What that achieves is it that the you want the new colour to blend with the old colour based on the opacity. This is the operation that ends up happening with this set up in pseudocode.

```c++
finalColour.rgb = newAlpha * newColour + (1 - newAlpha) * oldColour;
finalColour.a = newAlpha.a;
```

**VkPipelineColorBlendStateCreateInfo** takes in a array of structures for all the framebuffers and allows you to set blend constants that you can use for blend factors in the above calculations. This is what this would look like.

```c++
VkPipelineColorBlendStateCreateInfo colourBlending{};
colourBlending.sType = VK_STRUCTURE_TYPE_PIPELINE_COLOR_BLEND_STATE_CREATE_INFO;
colourBlending.logicOpEnable = VK_FALSE;
colourBlending.logicOp = VK_LOGIC_OP_COPY; //optional
colourBlending.attachmentCount = 1;
colourBlending.pAttachments = &colourBlendAttachment;
colourBlending.blendConstants[0] = 0.f; //optional
colourBlending.blendConstants[1] = 0.f; //optional
colourBlending.blendConstants[2] = 0.f; //optional
colourBlending.blendConstants[3] = 0.f; //optional
```

If you want to use this second type of blending the bitwase combination, you need to set the **.logicOpEnable** to VK_TRUE. You need to have need to have the **.pAttachments** which takes the colour blend operation via the **VkPipelineColorBlendAttachmentState** to have a value for it's **.blendEnable** VK_TRUE for at least some of the framebuffers or that blending wont happen. Then you can set the **.logicOp** to have that bitwase operation to blend.

In the same struct **VkPipelineColorBlendAttachmentState** the colour mask **.colorWriteMask** change what's being effected by this bitwase combination operation. In the above struct we've disabled both blending operations here.
##### Pipeline layout
This is were the shader **uniforms** and **push constants** are set up to be used. These are like global variables in the shader. Think about the transformation matrix and the texture samplers from a texture type thing. These are all things you can set up here.

If you want to use uniform you need to set them up with the **VkPIpelineLayout** vulkan type. To create that you need to use the creation struct **VkPipelineLayoutCreateInfo**. With no uniforms or push constants this is want you'll want for this set.

```c++
VkPipelineLayout pipelineLayout;
VkPipelineLayoutCreateInfo pipelineLayoutCreateInfo{};
pipelineLayoutCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pipelineLayoutCreateInfo.setLayoutCount = 0;
pipelineLayoutCreateInfo.pSetLayouts = nullptr;
pipelineLayoutCreateInfo.pushConstantRangeCount = 0;
pipelineLayoutCreateInfo.pPushConstantRanges = nullptr;

if (vkCreatePipelineLayout(device, &pipelineLayoutCreateInfo, nullptr, &pipelineLayout) != VK_SUCCESS) 
{
	assert(false);
	printf("Failed to create the layout pipeline.\n");
}
```

Clean up is also required.

```c++
vkDestroyPipelineLayout(device, pipelineLayout, nullptr);
```
##### Render passes
Now we have done the fixed function part of the graphics pipeline we need to tell vulkan about the framebuffer attachments that we will use while rendering. We need to set up a couple of things.
- The colour and depth buffers
- How many samples there be for each buffer
- How should we handle both the buffer and the samples for rendering operation.
All this information is set up in the **render pass**. 
##### Attachment descriptions
Attachments change depending on the buffers, the samples and what you want to do with them. Attachments are set up with a **VkAttachmentDescription**. For a simple rendering of triangle you just need to worry about 1 colour attachment.

```c++
VkAttachmentDescription colourAttachment{};
colourAttachment.format = swapchainFormat;
colourAttachment.samples = VK_SAMPLE_COUNT_1_BIT;
```

Here we set the format to the swapchain image format and we set the samples to **VK_SAMPLE_COUNT_1_BIT** because we aren't doing any sampling.

Attachments have **loadOp**, **storeOp**, **stencilLoadOp** and **stencilStoreOp** ones for stencil and the other one isn't. The loadOp and storeOp apply to colour and depth values and the stencil apply to stencil.

For **loadOp** we have these options:
- **VK_ATTACHMENT_LOAD_OP_LOAD** preserve the existing contents of the attachment.
- **VK_ATTACHMENT_LOAD_OP_CLEAR** clear the values to a constant at the start.
- **VK_ATTACHMENT_LOAD_OP_DONT_CARE** existing contents are undefined we don't care about them. 
For **storeOp** we have these options:
- **VK_ATTACHMENT_STORE_OP_STORE** rendered content will be stored in memory and can still be read later
- **VK_ATTACHMENT_STORE_OP_DONT_CARE** Contents fo the framebuffer will be undefined after rendering operation

In the case were you have a framebuffer colour attachment and the format is part of the swapchain you'll want to clear the **loadOp** to black. Meaning that the beginning of every frame you have a clean framebuffer to draw to.

For the **storeOp** we want to set this to be store because we are interesting in seeing the content on screen.

```c++
colourAttachment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
colourAttachment.storeOp = VK_ATTACHMENT_STORE_OP_STORE;
```

Since we aren't using any stencil data we can set **.stencilLoadOp** and **.stencilStoreOP** to the most memory efficent (for performance) DONT_CARE

```c++
colourAttachment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
colourAttachment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
```

Textures and framebuffer are just VkImage's which just contains pixel data. The images can be layed out in memory differently depending on the usage. The most common layouts you'll transition to are:
- **VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL** images are used as a colour attachment
- **VK_IMAGE_LAYOUT_PRESENT_SRC_KHR** this is used for when you want to present an image.
- **VK_IMAGE_LAYOUT_TRANSFER_DST_OPTIMAL** this is used when you need to do something like a memory copy operation.
These are more important when you need to start texturing objects. It's important to know that images need to be transitioned into a specific layout that's suitable for the type of operation they are needed to do next.

So for our colour attachment that's just meant to render a triangle and be presented to the screen after the VkImage has been drawn to. This is what you'll want to to transition to.

```c++
colourAttachment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
colourAttachment.finalLayout = VK_IMAGE_LAYOUT_PRESENT_SRC_KHR;
```

So this is for the render pass and the **.initialLayout** tells vulkan what the layout is before the render pass begins and the **.finalLayout** tells vulkan what needs to transitioned to when render pass finishes. 

Using the **VK_IMAGE_LAYOUT_UNDEFINED** value for the **.intialLayout** says we don't care what the the VkImage's layout was before this. If a VkImage is in this memory layout anything could happen to it, it shouldn't be considered as "cleared" but "unitialised memory" as the graphics card could anything to that piece of memory.
##### Subpasses and attachment references
A subpass sub divides render passes into seperate logical phases. The benefit of using mulitple subpasses is that the GPU can do some optimisations. 

A single render pass can have muitple subpasses. A subpass is a basically just a collection of rendering operation that are sequential that depend on the framebuffer previous passes. For example a sequence of post-processing effect that are applied to each other. 

If you group all these operations in a render pass vulkan will be able to reorder the operations which will conserve memory bandwidth for possible better performance. For drawing a triangle you need only one subpass.

Every subpass references one or more attachment that we've described above. The subpass needs another struct to actually reference the attachments correct called **VkAttachmentReference**.

```c++
VkAttachmentReference colourAttachmentRef{};
colourAttachmentRef.attachment = 0;
colourAttachmentRef.layout = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL;
```

The **.attachment** is an index that ties the subpass to the 
```glsl
layout(location = 0) out vec4 outColour
```
in the fragment shader. This is **EXTREMELY** important to make sure you match the indices up properly.

The **.layout** specifies which what type of layout attachment we would like to have for a given reference. Vulkan will automatically transition the attachment to whatever we specified in the layout, when the subpass starts. When using **VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL**
you're effectively making a colour buffer and when drawing a triangle that's the case.

Next we need to create the subpass with **VkSubpassDescription**

```c++
VkSubpassDescription subpass{};
subpass.pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS;
```

Vulkan in the future will support compute subpasses so for now we have to be explicit that we are using this for graphics. 

```c++
subpass.colorAttachmentCount = 1;
subpass.pColorAttachments = &colourAttachmentRef;
```

This is how the fragment shader knows that `layout(location = 0) out vec4 outColour` and the subpass line up, **for colour**. 

There are 4 others that match to a different attachments for shaders:
- **.pInputAttachment** This attachment is for reading things from a shader.
- **.pResolveAttachment** This attachment is for multisampling colour attachments.
- **.pDepthStencilAttachment** This attachment is for depth and stecil data.
- **.pPreserveAttachments** This attachment is for things that aren't used by the subpass but the data much be preserved. 
##### Creation of the render pass
When creating the render pass, you need to use the creation struct **VkRenderPassCreateInfo** which makes the **VkRenderPass** object. The creation struct takes array's of **VkAttachmentDescription**'s and **VkSubpass**'s. 

This is what a basic render pass set up looks like
```c++
VkRenderPass renderPass;
VkRenderPassCreateInfo renderPassCreateInfo{};
renderPassCreateInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_CREATE_INFO;
renderPassCreateInfo.attachmentCount = 1;
renderPassCreateInfo.pAttachments = &colourAttachment;
renderPassCreateInfo.subpassCount = 1;
renderPassCreateInfo.pSubpasses = &subpass;

if (vkCreateRenderPass(device, &renderPassCreateInfo, nullptr, &renderPass) != VK_SUCCESS) 
{
	printf("Failed to create a render pass.");
}
```

Deletion is done in the same was a normal

```c++
vkDestroyRenderPass(device, renderPass, nullptr);
```
##### Completing the graphics pipeline.
To complete the graphics pipeline we need to combine all the structures in the **VkGraphicsPipelineCreateInfo** to complete things. Here are the main cliff notes on what you need to do to complete this.
- **Shader stages**: You need the shader modules that define the functionality of the programmable stages of the graphics pipeline.
- **Fixed function states:** You need all the structures that set up the fixed function graphics pipeline stuff.
- **Pipeline Layout**: which sets up the uniforms and the push constants you'll be using for your shaders.
- **Render pass**: These are the attachment referenced by the pipeline stages and their usage.

Remember don't delete the shader modules with **vkDestroyShaderModule** before you create this graphics pipeline.

We start off with setting the shader stage and their count.
```c++
VkGraphicsPipelineCreateInfo pipelineInfo{};
pipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
pipelineInfo.stageCount = 2;
pipelineInfo.pStages = shaderStages;
```

Then you'll want all the fixed function stuff
```c++
pipelineInfo.pVertexInputState = &vertexInputState;
pipelineInfo.pInputAssemblyState = &inputAssembly;
pipelineInfo.pViewportState = &viewportCreateInfo;
pipelineInfo.pRasterizationState = &rasteriser;
pipelineInfo.pMultisampleState = &multisamplingState;
pipelineInfo.pDepthStencilState = nullptr; //TODO add depth and stencil support
pipelineInfo.pColorBlendState = &colourBlending;
pipelineInfo.pDynamicState = &dynamicState;
```

Then you want the pipeline layout.
```c++
pipelineInfo.layout = &pipelineLayout;
```

Then you want to add your render pass with the subpass index of 0.
```c++
pipelineInfo.renderPass = renderPass;
pipelineInfo.subpass = 0;
```

You can set things up to work with more than 1 render pass but they all have to be compatible with each other.

Then you can create everything with the **vkCreateGraphicsPipelines**

```c++
VkPipeline mainPipeline{};
if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 1, &pipelineInfo, nullptr, &mainPipeline) != VK_SUCCESS) 
{
	assert(false);
}
```

Now the parameters are a little more detailed here. First you have the second argument which takes in a **VkPipelineCache** value. This is really useful if you need to recreate a pipeline you've already made before. For example, if you're running a game and you make all the graphics pipelines needed you actually save them to disk and load this caches next time saving pipeline create time.

The rest of the arguments are as list
1) The logical device
2) The pipeline cache value which can be null.
3) The amount of **VkGraphicsPipelineCreateInfo** you are passing in.
4) The **VkGraphicsPipelineCreateInfo** themselves
5) The vulkan allocator callback
6) The final **VkPipeline**

The destruction is pretty straight forward, you just need to destroy this before anything in the pipeline is destroy.

```c++
vkDestroyPipeline(device, mainPipeline, nullptr);
```
##### Derived graphics pipelines
There are two new parameters **.basePipelineHandle** and **.basePipelineIndex** with these you can make graphics pipelines that are derived from an existing pipeline. The upside of this is that if they share common functionality it's faster to create a graphics pipeline, and it's faster to switch between child and parent graphics pipelines.

To do this you either use **.basePipelineHandle** to refer to a graphics pipeline object or **.basePipelineIndex** to refer to a graphics pipeline with an index. 

For both the parent and the child graphics pipeline the **.flag** setting needs to be **VK_PIPELINE_CREATE_DERIVATIVE_BIT** of the VkGraphicsPipelineCreateInfo struct. The child needs all the mandatory just like it's parent. 

If you want to do this via the **.basePipelineIndex** you have to create the parent and child together in one function call. If you want to use the **.basePipelineHandle** you have to create parent first. You have to use one **OR** the other. You can't use both. you have to turn off other version by setting **.basePipelineHandle** to VK_NULL_HANDLE or the **.basePipelineIndex** to -1
 
This the index version set up.

```c++
VkGraphicsPipelineCreateInfo pipelineInfo{};
pipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
pipelineInfo.stageCount = 2;
pipelineInfo.flags = VK_PIPELINE_CREATE_ALLOW_DERIVATIVES_BIT;
pipelineInfo.pStages = shaderStages;
pipelineInfo.pVertexInputState = &vertexInputState;
pipelineInfo.pInputAssemblyState = &inputAssembly;
pipelineInfo.pViewportState = &viewportCreateInfo;
pipelineInfo.pRasterizationState = &rasteriser;
pipelineInfo.pMultisampleState = &multisamplingState;
pipelineInfo.pDepthStencilState = nullptr;
pipelineInfo.pColorBlendState = &colourBlending;
pipelineInfo.pDynamicState = &dynamicState;
pipelineInfo.layout = pipelineLayout;
pipelineInfo.renderPass = renderPass;
pipelineInfo.subpass = 0;
pipelineInfo.basePipelineHandle = VK_NULL_HANDLE;
pipelineInfo.basePipelineIndex = 1;

VkGraphicsPipelineCreateInfo childPipelineInfo{};
childPipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
childPipelineInfo.flags = VK_PIPELINE_CREATE_DERIVATIVE_BIT;
childPipelineInfo.stageCount = 2;
childPipelineInfo.pStages = shaderStages;
childPipelineInfo.pVertexInputState = &vertexInputState;
childPipelineInfo.pInputAssemblyState = &inputAssembly;
childPipelineInfo.pViewportState = &viewportCreateInfo;
childPipelineInfo.pRasterizationState = &rasteriser;
childPipelineInfo.pMultisampleState = &multisamplingState;
childPipelineInfo.pDepthStencilState = nullptr;
childPipelineInfo.pColorBlendState = &colourBlending;
childPipelineInfo.pDynamicState = &dynamicState;
childPipelineInfo.layout = pipelineLayout;
childPipelineInfo.renderPass = renderPass;
childPipelineInfo.subpass = 0;
childPipelineInfo.basePipelineHandle = VK_NULL_HANDLE;
childPipelineInfo.basePipelineIndex = pipelineInfo.basePipelineIndex - 1;

VkGraphicsPipelineCreateInfo structs[] = { pipelineInfo, childPipelineInfo };

VkPipeline pipelines[2]{};
if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 2, pipelines, nullptr, childPipeline) != VK_SUCCESS)
{
	assert(false);
	printf("Failed to create pipeline.");
}
```

You have to set the child's **.basePipelineIndex** to one minus the parent's **.basePipelineIndex** with the line `childPipelineInfo.basePipelineIndex = pipelineInfo.basePipelineIndex - 1;`

Then you create both pipelines using array's instead of seperately. 

Creation of the using VkPipeline reference is much simpiler
```c++
VkGraphicsPipelineCreateInfo pipelineInfo{};
pipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
pipelineInfo.flags = VK_PIPELINE_CREATE_ALLOW_DERIVATIVES_BIT;
//... All the pipeline creation
pipelineInfo.basePipelineHandle = VK_NULL_HANDLE;
pipelineInfo.basePipelineIndex = -1;

VkPipeline mainPipeline;
if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 1, &pipelineInfo, nullptr, &mainPipeline) != VK_SUCCESS) 
{
	assert(false);
}

VkGraphicsPipelineCreateInfo childPipelineInfo{};
childPipelineInfo.sType = VK_STRUCTURE_TYPE_GRAPHICS_PIPELINE_CREATE_INFO;
childPipelineInfo.flags = VK_PIPELINE_CREATE_DERIVATIVE_BIT;
//... All the pipeline creation
childPipelineInfo.basePipelineHandle = mainPipeline;
childPipelineInfo.basePipelineIndex = -1;
	
VkPipeline childPipeline;
if (vkCreateGraphicsPipelines(device, VK_NULL_HANDLE, 1, &childPipelineInfo, nullptr, &childPipeline) != VK_SUCCESS)
{
	assert(false);
}
```

We actually don't care about the parent index or handle here, these can be ignored. The rest of the code I think in explain everything. Just look at the **.basePipelineHandle** in the childPipelineCreateInfo to see the different.
##### Framebuffers
The framebuffers set up is very much dependent on how you set up the render pass, as the render pass handles the framebuffer.

Attachments specified during the render pass are bound by wrapping them into a VkFramebuffer object. So the framebuffer contains all the VkImageView object that we created in the render pass via the **VkAttachmentDescription**. 

For each swapchain image you'll want to create a framebuffer for it. This will contain the render pass, subpass, attachment description and reference.

The framebuffer images after set up are used within a command. When you're trying to use the render pass to do the drawing operations you need to give the render pass the framebuffer image it's going to use, from the list of framebuffers you have already created. 

The image index that selects the correct framebuffer is controlled by the swapchain. It tells us what image index is safe to draw to as the swapchain operates. This is all done at draw time.

The setup for the framebuffers looks something like
```c++
std::vector<VkFramebuffer> swapchainFramebuffers(swapchainImageViews.size());

for (uint32_t i = 0; i < swapchainImageViews.size(); ++i) 
{
	VkImageView attachments[] =
	{
		swapchainImageViews[i]
	};
	
	VkFramebufferCreateInfo framebufferInfo{};
	framebufferInfo.sType = VK_STRUCTURE_TYPE_FRAMEBUFFER_CREATE_INFO;
	framebufferInfo.renderPass = renderPass;
	framebufferInfo.attachmentCount = 1;
	framebufferInfo.pAttachments = attachments;
	framebufferInfo.width = swapchainExtent.width;
	framebufferInfo.height = swapchainExtent.height;
	framebufferInfo.layers = 1;
	
	if (vkCreateFramebuffer(device, &framebufferInfo, nullptr, &swapchainFramebuffers[i]) != VK_SUCCESS)
	{
		assert(false);
	}
}
```

So we planning on using this framebuffer a draw time with the render pass at draw time, the **.renderPass** need to match the render pass we are going to use at draw time.

The **.pAttachments** are the swapchain images. Note we only have 1 attachment description which described the a colour image. So when we are making attachments we wrap things up in `attachments[]` as in the future we might a tonne more attachments descriptions. The **.attachmentCount** is the totally number of attachments we have in our render pass and framebuffer.

When setting up the the framebuffer with a basic swapchain that contains a single image per image, you want set the **.layers** to 1.

Deletion is pretty straight forward.

```c++
for (uint32_t i = 0; i < swapchainFramebuffers.size(); ++i)
{
	vkDestroyFramebuffer(device, swapchainFramebuffers[i], nullptr);
}
```
##### Command buffers
You can't individually run commands in vulkan you need recorded and ran in bulk. These are recorded on command buffer objects. The advantage of this is that Vulkan and efficiently rework things to be more efficent. You can also record commands on command buffer on different threads if you want.

Command buffer are executed by submitting them every frame to the device queue. Such a the graphics or presentation queue. 
##### Command pools
You'll need to create a command pool before you can create the command pools. The command pools manages the memory of the command buffers that are allcoated from them. 

```c++
VkCommandPoolCreateInfo poolCreateInfo{};
poolCreateInfo.sType = VK_STRUCTURE_TYPE_COMMAND_POOL_CREATE_INFO;
poolCreateInfo.flags = VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT;
poolCreateInfo.queueFamilyIndex = mainQueueFamilyIndex;
```

There are two flags for **.flags**
- **VK_COMMAND_POOL_CREATE_TRANSIENT_BIT** This says that the buffers are rerecorded with new commands very often, which means that there might be a lot allocation behaviour.
- **VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT** This command buffers to be rerecorded individually, without this flag they all have be reset together.
If you want to record the command buffer you want the **VK_COMMAND_POOL_CREATE_RESET_COMMAND_BUFFER_BIT** as you'll be able to reset it and rerecord it.

Command buffer are executed by submitting them every frame to the device queue. Such a the graphics or presentation queue. So in the **.queueFamilyIndex** we have choosen to use the **mainQueueFamilyIndex** which has been used to create the graphics device queue.

Creation and destruction are pretty straight forward.
```c++
VkCommandPool commandPool;
if (vkCreateCommandPool(device, &poolCreateInfo, nullptr, &commandPool))
{
	printf("Failed to create a command pool.");
}
```

```c++
vkDestroyCommandPool(device, commandPool, nullptr);
```

The command pool is going to be used throughout the program so it needs to be delete at the end.
###### Command buffer allocation
Creating the command buffer object with **VkCommandBuffer** will be automatically destroyed with the command pool.

To allocate command buffer you use the specialised function **vkAllocateCommandBuffers** along with a creation struct. 

```c++
VkCommandBufferAllocateInfo allocateInfo{};
allocateInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
allocateInfo.commandPool = commandPool;
allocateInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
allocateInfo.commandBufferCount = 1;

VkCommandBuffer commandBuffer;
if (vkAllocateCommandBuffers(device, &allocateInfo, &commandBuffer)) 
{
	printf("Failed to create a command buffer.");
}
```

You have two different levels of command buffers in vulkan that get set with **.level**. These are:
-  **VK_COMMAND_BUFFER_LEVEL_PRIMARY** These **CAN** be submitted to a queue for execution but cannot call other command buffers.
- **VK_COMMAND_BUFFER_LEVEL_SECONDARY** These **CAN'T** be submitted directly, but can be called from primary command buffers.

Secondary command buffer usage helps with command reuse which can help with performance. They can't however have anything to do with presentation commands as these are things that just get submitted.
###### Recording commands
Recording a command buffer is all about writing commands into the command buffer to be executed later. With a graphics command buffer you'll need to record an image to draw into, whether than that be a separate VkImage or one straight from the swapchain.

This should be happening at the beginning of frame but not necessarily rerecorded every frame. You create the command buffer with the fuction **vkBeginCommandBuffer** and with the struct **VkCommandBufferBeginInfo**.

```c++
VkCommandBufferBeginInfo beginInfo{};
beginInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_BEGIN_INFO;
beginInfo.flags = 0;
beginInfo.pInheritanceInfo = 0;

if (vkBeginCommandBuffer(commandBuffer, &beginInfo) != VK_SUCCESS) 
{
	printf("Failed to begin recording command buffer.");
}
```

The **.flags** parameter specified how we're going to use the command buffer. These values can be:
- **VK_COMMAND_BUFFER_USAGE_ONE_TIME_SUBMIT** The command buffer will be rerecorded right after exection.
- **VK_COMMAND_BUFFER_RENDER_PASS_CONTINUE_BIT** This is used for the secondary command buffer will be used entirely within a single render pass.
- **VK_COMMAND_BUFFER_USAGE_SIMULTANEOUS_USE_BIT** This is for a command buffer that is resubmitted while it is already pending for execution.

**.pInheritanceInfo** is used for secondary command buffers. It specifies which state to inherit from calling primary command buffers.

**vkBeginCommandBuffer** clears the previously recorded command buffer, you can't append command buffers.
##### Starting a render pass
To start drawing you'll need to begin the render pass using the **vkCmdBeginRenderPass** using the **VkRenderPassBeginInfo** struct.

```c++
VkRenderPassBeginInfo renderPassBeginInfo{};
renderPassBeginInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_BEGIN_INFO;
renderPassBeginInfo.renderPass = renderPass;
renderPassBeginInfo.framebuffer = swapchainFramebuffers[imageIndex];
```

**.renderPass** is the render pass with the attachments description and references along with the subpass.

The idea here with **.framebuffer** using the swapchain images we created eariler we aquire the image index value with the uint32_t **imageIndex** from the swapchain image, to directly draw on to. You may need to do something differently if you're not directly drawing on to the swapchain images.

Then we have to tell vulkan the areas were the load operations and and the store operation which get set up with the attachment description which are bundled with the render pass.

```c++
renderPassBeginInfo.renderArea.offset = { 0, 0 };
renderPassBeginInfo.renderArea.extent = swapchainExtent;
```

Since we are drawing straight onto a swapchain image we just take thoughs, pixels outside this range are undefined. You want your attachment description bounds to be the same as the **.renderArea** for better performance.

```c++
VkClearValue clearColour = { { { 0.218f, 0.f, 0.265f, 1.f } } };
renderPassBeginInfo.clearValueCount = 1;
renderPassBeginInfo.pClearValues = &clearColour;
```

Remember when we set the **.loadOp** in the VkAttachmentDescription to be **VK_ATTACHMENT_LOAD_OP_CLEAR** clearing the screen for a basic colour attachment? Well this is what the colour attachmen clears to. It's better to not make it black because you wont know if you're clearing is just working hence this dark purple. NOTE: Here we aren't clearing the depths.

```c++
vkCmdBeginRenderPass(commandBuffer, &renderPassBeginInfo, VK_SUBPASS_CONTENTS_INLINE);
```

With this command the render pass begins. This is a command because you can tell with the **vkCmd** part of the function. These return void and there is no error checking with them.

Commands in vulkan always take the command buffer to which the command you've written gets recorded to. The last paramter is interesting because that defines how the drawing commands will be provided.
- **VK_SUBPASS_CONTENTS_INLINE** The commands will be in the primary command buffer, there will be no secondary command buffer. 
- **VK_SUBPASS_CONTENTS_SECONDARY_COMMAND_BUFFERS** The render pass be will executed from a secondary command buffer.
##### The drawing commands

Before drawing anything you'll want to bind a graphics pipeline. 

```c++
vkCmdBindPipeline(commandBuffer, VK_PIPELINE_BIND_POINT_GRAPHICS, mainPipeline);
```

The second argument tells vulkan what type of pipeline are you binding. Which tells vulkan what attachment to use in the fragment shader which we set up with the **VkAttachmentReference**. It also tells vulkan what type of operations are we doing to use.

Normally you'll want to set up the viewport and scissor if you've made them dynamic when creating the graphics pipeline.

```c++
VkViewport viewport{};
viewport.x = 0.f;
viewport.y = 0.f;
viewport.width = float(swapchainExtent.width);
viewport.height = float(swapchainExtent.height);
viewport.minDepth = 0.f;
viewport.maxDepth = 1.f;
vkCmdSetViewport(commandBuffer, 0, 1, &viewport);

VkRect2D scissor{};
scissor.offset = { 0, 0 };
scissor.extent = swapchainExtent;
vkCmdSetScissor(commandBuffer, 0, 1, &scissor);
```

This basically the same way you would set up the scissor and the viewport if you were making it statically in the pipeline set up.

After all that you can simply run the draw command, this is for drawing a triangle. Note it's not index because we have no index buffer.
```c++
vkCmdDraw(commandBuffer, 3, 1, 0, 0);
```

The arguments are after the command buffer:
- **vertexCount** This is for the amount of vertices you want to render. 
- **instanceCount** This is for used fro instanced rendering. We have 1 instance so it's 1.
- **firstVertex** This is the offset into a vertex buffer. This defined the lowest value of **gl_VertexIndex**
- **firstInstance** This is used for instance redering and defines the lowest value of **gl_Instance**

Finally you'll want to end the render pass with
```c++
vkCmdEndRenderPass(commandBuffer);
if (vkEndCommandBuffer(commandBuffer) != VK_SUCCESS) 
{
	printf("Failed to record a command buffer");
}
```

##### Rendering and presentation
When you actually want to render a frame you need to do these things:
1) Wait for the previous frame to finish.
2) Acquire an image from the swapchain.
3) Record a command buffer which draws the scene onto that image.
4) Submit the recorded command buffer
5) Present the swapchain image.

This is the overview you need to keep in mind while other things are mentioned.

##### Synchronisation
With synchronisation everything explicit meaning that order of operation is up to us. We need to tell the driver what to do with all the synchronisation primitive. It's important to note that the GPU and CPU are asynchronous, meaning that a lot of vulkan functions will return before the work on the GPU is done.

There are 3 major things with need to synchronise
- Acquire an image from the swap chain.
- Execute commands that draw onto the acquired image.
- Present that image to the screen for presentation, returning it to the swapchain.

This is great but the order of this function finishing on the GPU is undefined, so you'll need to wait for them to finish to get everything working correctly.
##### Semaphores
Semaphores are used order between operations. These are used to order commands within a queue and synchronise different queues together. There is also functions outside of commands that also need semaphores.

There are two types of semaphores **binary** and **timeline**. To use timeline you need an extension to get this working. Binary semaphores are the simpliest and require less set up but can do less. Here I will only talk about binary semaphores as they are simplest.

A binary semaphore is either unsignalled or signalled. It begins as unsignaled. This works by one command in a queue signalling the semaphore when the command is complete, another command will wait on that semaphore until it is singled to go on and do it's worked. Once the second command has started the semaphore goes back to unsignaled.
1) Semaphore unsignalled
2) Command one begins running and will signal **AFTER** it's done.
3) Command two wait for that signal in the semaphore.
4) Command two runs when the signal is received.
5) Semaphore automatically goes back to unsignalled after command two **STARTS**.

With the semaphores the synchronisation is **ONLY** for commands running that run on the GPU. The CPU is free to continue on while semaphores are doing their thing.
##### Fences
Fences are used to synchronise between the host (CPU) and the GPU. If host needs to know when the GPU is done we use fences.

Fences work similarly to binary semaphores. They are either signalled or unsignalled. When work is finished with a fense we can signal it which means the host can continue on because we know the GPU is up to date.

If you can avoid fences it's best to because the CPU will just wait until it gets a signal from the fence which isn't useful work. You also need to reset fences manually. 
##### What to choose?
You'll need a fence to pause the host, to make sure that we are rendering 1 image because we can't rerecord command buffers this would overwrite the command buffer while it's being used.

You'll need two semaphores to order the operations that are happening with in the swapchain. One for acquiring the next image that's ready to be draw to. Another semaphore for making sure that we wait for things to finish rendering, before we present the image to the screen.
##### Setting up the synchronisation objects.
So to handle drawing to a screen we will need 2 semaphores and 1 fence.

```c++
VkSemaphore imageAvailableSemaphore;
VkSemaphore renderFinishSemaphore;
VkFence framesInFlight;

VkSemaphoreCreateInfo semaphoreInfo{};
semaphoreInfo.sType = VK_STRUCTURE_TYPE_SEMAPHORE_CREATE_INFO;

VkFenceCreateInfo fenceInfo{};
fenceInfo.sType = VK_STRUCTURE_TYPE_FENCE_CREATE_INFO;
fenceInfo.flags = VK_FENCE_CREATE_SIGNALED_BIT;

if (vkCreateSemaphore(device, &semaphoreInfo, nullptr, &imageAvailableSemaphore) != VK_SUCCESS)
{
		printf("Failed to create image available semaphore");
}

if (vkCreateSemaphore(device, &semaphoreInfo, nullptr, &renderFinishSemaphore) != VK_SUCCESS)
{
	printf("Failed to create render finished semaphore");
}

if (vkCreateFence(device, &fenceInfo, nullptr, &framesInFlight) != VK_SUCCESS)
{
	printf("Failed to create the fence.");
}
```

The only important part is the **VkCreateFenceInfo** **.flags** this is set to **VK_FENCE_CREATE_SIGNALED_BIT** meaning we set up the fence to be signalled from the start. You want to do this because we are waiting for the fence to be signled before we begin our next frame. Well what about the first frame? There wasn't a previous frame to wait for. If we don't set up the fence at the beginning we will dead locked immediately. 

Destroying them is pretty simple too.

```c++
vkDestroySemaphore(device, imageAvailableSemaphore, nullptr);
vkDestroySemaphore(device, renderFinishSemaphore, nullptr);
vkDestroyFence(device, framesInFlight, nullptr);
```
##### Waiting for the previous frame.
At the beginning of the drawing routine you'll need to wait for the previous frame.  You do this with the **vkWaitForFence**

```c++
vkWaitForFences(device, 1, &framesInFlight, VK_TRUE, UINT64_MAX);
```

The **vkWaitForFence** can wait more muiltple fences. The secound argument is the amount of fences in the array, the next is the VkFence array and the **VK_TRUE** here is a bool that says should the host wait for all the fences. Since we are using 1 fence this doesn't matter to use a lot. The last argument is the timeout, using UINT64_MAX pretty much disables this wait timing out. 

Remember you have to reset the fence manually so after the wait is over from **vkWaitForFences** we need to reset the fence back it's unsignlled state with.

```c++
vkResetFences(device, 1, &framesInFlight);
```

##### Acquirng an image from the swapchain.
Next we need to get the image from the swapchain. 

```c++
uint32_t imageIndex;
vkAcquireNextImageKHR(device, swapchain, UINT64_MAX, imageAvailableSemaphore, VK_NULL_HANDLE, &imageIndex);
```

The first two parameters are telling vulkan were we get the image from. The third argument is the timeout for this command which we disable again with the UINT64_MAX.

The 4th and 5th argument are to do with waiting for the presentation engine to finish it's work. We are only using a semaphore which will be signalled when the presentation is done. 

The last argument tells us which VkImage is usable in the swapchain array. We use this value to later use to access the correct swapchain image in our render pass as the framebuffer.
##### Submitting the command buffer
Queue submission and synchronisation is configured with **VkSubmitInfo**. 

```c++
VkSubmitInfo submitInfo{};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;

VkSemaphore waitSemaphores[] = { imageAvailableSemaphore };
VkPipelineStageFlags waitStages[] = { VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT };
submitInfo.waitSemaphoreCount = 1;
submitInfo.pWaitSemaphores = waitSemaphores;
submitInfo.pWaitDstStageMask = waitStages;
```

What we are doing here is setting up part of the pipeline stages that need to wait. We need to wait on the **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** This is the part of that handles the output of the final colour. This stop things writing to an image until it is ready.

This means that we can run the vertex shader and other stages of the pipeline before we have to wait. We explain why we are doing this in the subpass dependency section.

```c++
submitInfo.commandBufferCount = 1;
submitInfo.pCommandBuffers = &commandBuffer;
```

Here we submit the command buffer for execution.

```c++
VkSemaphore signalSemaphores[] = { renderFinishSemaphore };
submitInfo.signalSemaphoreCount = 1;
submitInfo.pSignalSemaphores = signalSemaphores;
```

This is simply the set up to signal the semaphore when the command buffer has finished executing.

```c++
if (vkQueueSubmit(mainQueue, 1, &submitInfo, framesInFlight) != VK_SUCCESS) 
{
	printf("Failed to submit the command buffer to draw with the current queue.");
}
```

For this function we have
- **mainQueue** which is queue we are submitting the command buffer on.
- **1** which is the total count of **VkSubmitInfo** in an array.
- **&submitInfo** is the struct itself or an array of them. Having an array can help with efficiency if the work load is large enough.
- **frameInFlight** is our fence (which is optional) this will signal the fence to wait for the command buffer to finish executing. This is so our frames don't over draw each other causing all sort of issues.
##### Setting up subpass dependencies
The later brings up subpass dependences which we need to solve. This is done with the **VkSubpassDependency** struct which needs to be set up as part of the render pass creation.

```c++
VkSubpassDependency dependency{};
dependency.srcSubpass = VK_SUBPASS_EXTERNAL;
dependency.dstSubpass = 0;
```

The first two fields specify the indices of the dependency and dependend subpass. 

The indices of the dependencies work so that the **.srcSubpass** takes in a index that is **SMALLER** than the **.dstSupass** (unless **VK_SUBPASS_EXTERNAL** is used). These connect other subpass dependencies together. 

**VK_SUBPASS_EXTERNAL** tells if the dependency starts at the beginning or end of the render pass depending if it's assigned to **.srcSubpass** (which is the beginning) or the **.dstSubpass** (which is the end). You can image a subpass depedency chain being set up like this:

```c++
VkSubpassDependency dependencyOne{};
dependencyOne.srcSubpass = VK_SUBPASS_EXTERNAL;
dependencyOne.dstSubpass = 0;

VkSubpassDependency dependencyTwo{};
dependencyTwo.srcSubpass = 0;
dependencyTwo.dstSubpass = 1;
```

Next we set up what pipeline stages and image attachments we are waiting. When we are using the **.src** paramters we are telling vulkan that **THIS** subpass need to wait on before any operations can begin within that subpass. The **.dst** is all about what needs to be completed before we can pass over this over to the NEXT subpass.

Here we only using 1 subpass within a render pass so these are self dependences. 

```c++
dependency.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
dependency.dstStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
```

For the **.srcStageMask** stage dependencence we are waiting on the swapchain to finish reading the image before we can access it from the previous frame, which is done with the **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** then we can begin writing the final colour of the frame to the swapchain's framebuffer's image.

For **.dstStageMask** the subpass and we need to have completed any work that involves the  **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** which in our case is writing our final colours to the swapchain's framebuffer's image.

```c++
dependency.srcAccessMask = 0;
dependency.dstAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT;
```

For **.srcAccessMask** we don't care what we came in as for our attachments so it's zero. The second one is making sure that subpass attachment has transitioned to a **VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT** before we move on. Meaning we have written to the image.

```c++
renderPassCreateInfo.dependencyCount = 1;
renderPassCreateInfo.pDependencies = &dependency;
```

Then when create the render pass we set up the subpass dependencies for our 1 subpass. 
##### Setting the sychronisation with the subpass dependencies
Without this synchronisation when submitting our command buffer we are assuming that the memory transition goes from undefined to colour attach optimial at the beginning of the render pass, this is WRONG. We haven't acquired the image yet to begin this transition.

This is what this code is set up to do.
```c++
VkSubmitInfo submitInfo{};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;

VkSemaphore waitSemaphores[] = { imageAvailableSemaphore };
VkPipelineStageFlags waitStages[] = { VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT };
submitInfo.waitSemaphoreCount = 1;
submitInfo.pWaitSemaphores = waitSemaphores;
submitInfo.pWaitDstStageMask = waitStages;
```

There are actually two way to wait for the image, in the **waitStages[]** array
1) If you set a semaphore the flag is set to **VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT** the ensure that the render passes don't begin until the image is available, avoiding setting up the subpass dependencies. This blocks the pre-colour outputting stages of the graphics pipeline.
2) If you set a semaphore the flag is set to **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** which forces the render pass to wait until this part of the graphics pipeline has been finished. However you need to have set up the subpass dependencies within the render pass correctly for this to work, which you should have already done to match the semaphore.
##### Presentation
If you've done everything needed to take the image to the submit your command buffer, the final to is tell the swapchain to submit your image to the screen. This is done with the **VkPresentInfoKHR** struct.

This code should all go at the end of the drawing a frame.

```c++
VkPresentInfoKHR presentInfo{};
presentInfo.sType = VK_STRUCTURE_TYPE_PRESENT_INFO_KHR;
presentInfo.waitSemaphoreCount = 1;
presentInfo.pWaitSemaphores = signalSemaphores;
```

Here we are waiting on the signalSemaphore which was used when we submitted the command buffer. Meaning we are waiting for everything to render and write that result out.

```c++
VkSwapchainKHR swapchains[] = { swapchain };
presentInfo.swapchainCount = 1;
presentInfo.pSwapchains = swapchains;
presentInfo.pImageIndices = &imageIndex;
presentInfo.pResults = nullptr;
```

Here we actually present the specify swapchain with **.pSwapchains** to present the images. The index of the image also needs to be presented with **.pImageIndices**. **.pResults** takes an array of VkResults which is useful if your using multiple swapchains because it will tell you if the images were presented successfully.

```c++
vkQueuePresentKHR(mainQueue, &presentInfo);
```

All this function is tell the swapchain to present the image to the screen. 

When closing you need to remember that GPU is asynchronous from the CPU meaning that operations on the GPU might be still happening so you'll need to add

```c++
vkDeviceWaitIdle(device);
```

all to pause operations. This is a very creud way of performaning synchronisations between the GPU and CPU but while closing this is enough.
##### Frame in flight
Frames in flight is to do with how many frames are currently processing at the same time. In newer version of vulkan you can't just handle 1 frame at a time if your swapchain image count is more than 1.

Earlier on when you had extracted the minimum and maximum amount of swapchain images through checking it's capability. We can that we had this number of images.

```c++
uint32_t imageCount = surfaceCapabilities.minImageCount + 1;
```

This is the totally number of frame in flight you should have. 

Since we want to process these frames at the same time we also need to duplicated the command buffer and the synchronisation object.

This is what creation would look like the command buffer.
```c++
std::vector<VkCommandBuffer> commandBuffers(imageCount);
VkCommandBufferAllocateInfo allocateInfo{};
allocateInfo.sType = VK_STRUCTURE_TYPE_COMMAND_BUFFER_ALLOCATE_INFO;
allocateInfo.commandPool = commandPool;
allocateInfo.level = VK_COMMAND_BUFFER_LEVEL_PRIMARY;
allocateInfo.commandBufferCount = uint32_t(commandBuffers.size());

if (vkAllocateCommandBuffers(device, &allocateInfo, commandBuffers.data())) 
{
	printf("Failed to create a command buffer.");
}

std::vector<VkSemaphore> imageAvailableSemaphore(imageCount);
std::vector<VkSemaphore> renderFinishSemaphore(imageCount);
std::vector<VkFence> framesInFlight(imageCount);
```

Since you know how to use flat arrays I'm sure you figure out the reset.

To do the rest we need to keep track of the current frame at run time. 

```c++
uint32_t currentFrame = 0;
bool running = true;
while(running)
{
	///... All event handling code here.
	
	vkWaitForFences(device, 1, &framesInFlight[currentFrame], VK_TRUE, UINT64_MAX);
	vkResetFences(device, 1, &framesInFlight[currentFrame]);

	//... All other drawing code here.

	currentFrame = (currentFrame + 1) % maxFramesInFlight;
}
```


The **currentFrame** should be set to 0 and index into command buffers, semaphores and fences as each frame needs it's own seperate synchronisation objects and command buffer. The last line caps the the current to cycle through 0, to n, where n is equal to how many images are in the swapchain.
##### Swapchain recreation
Note before I start I did things slightly differently from the tutorial the information is sourced from but the information is sourced from the tutorial.

There is an issue that if not dealt with can cause some issues. If you resize the the window the window surface changes which means the swapchain and surface aren't compatible anymore. So event were the window surface changes we need to recreate the swapchain.

To recreate the swapchain you'll need to reconstruct everything to do with the swapchain, the VkImageViews that need to be created for each swapchain image, and the framebuffers that use those swapchain images to be created.

It's important to note that you might want to even create the renderpass. It's possible that a monitor has lower DPI and then the window gets dragged to a higher DPI monitor.

```c++
static void recreateSwapchain()
{
    vkDeviceWaitIdle(device);

    vkGetPhysicalDeviceSurfaceCapabilitiesKHR(physicalDevice, surface, &surfaceCapabilities);
    swapchainExtent = surfaceCapabilities.currentExtent;

    if(swapchainExtent.width == 0 || swapchainExtent.height == 0) 
    {
        return;
    }

    cleanupSwapchain();

    createSwapchain();
    createImageViews();
    createFramebuffers();
}
```

First thing we need to do is wait for the GPU with **vkDeviceWaitIdle(device);** It's not a good idea ever to change resources still in use so when we are recreating the swapchain we are doing to recerate the VkImageViews which might be in use somewhere.

The next thing we do is guard against the viewport degrading to 0 height or width. You can't when recreating the swapchain because we are not allowed to create a swapchain with an **.imageExtent** in the **VkSwapchainCreateInfoKHR** creation struct with either 0 in height or width. That's why we are quering the **surfaceCapabilities** early which will be used in the **createSwapchain();** function if we aren't at 0 height or width.

The 0 height and width also guards against minimised windows.

Next we have to destroy the old framebuffers, old swapchain ImageView's and the swapchain, all before we start recreating them. This is done the **cleanupSwapchain();**

```c++
static void cleanupSwapchain() 
{
    for (auto framebuffer : swapchainFramebuffers)
    {
        vkDestroyFramebuffer(device, framebuffer, nullptr);
    }

    for (auto imageView : swapchainImageViews)
    {
        vkDestroyImageView(device, imageView, nullptr);
    }

    vkDestroySwapchainKHR(device, swapchain, nullptr);
}
```

The reset of the functions all are part of the swapchain create which you can read about here

Creating the swapchain happens in the **createSwapchain();** below is everything you need to write that function.
[[What is a swapchain]]
[[Checking surface and swapchain compability]]
[[Setting up the swapchain extension]]
[[Querying surface capabilities]]
[[Checking surface format]]
[[Selecting a presentation mode]]
[[Creating the swapchain]]
[[Getting swapchain images]]

Creation of the swapchain image views is done in the **createImageViews();** below is everything you need to write that function.
[[What are image views]]
[[Creating an image view]]
[[Destroying a image view]]
 
Creation of the framebuffers which come from the swapchain are in the **createFramebuffers();** below is everything you need to write that function.
[[What is a framebuffer]]
[[Creating the framebuffers]]

You can do better than this because we aren't going to use the **.oldSwapchain** feature in the creation of the new swapchain. This allows us to still deal with frames in flight while the swapchain is being recreated. 
##### Suboptimal or out-of-date swapchain.
If you have a function that recreate a swapchain now you just need to know were to apply it. This should be applied when you **vkAcquireNextImageKHR** and when you submit things to be presented with the function **vkQueuePresentKHR**. These both return values you can check for:
1) **VK_ERROR_OUT_OF_DATE_KHR** The swapchain become incompatible with the surface and ca no longer be used for rendering. This typical means the window has window resize.
2) **VK_SUBOPTIMAL_KHR** The swapchain can still be used successfully present to the surface, but the swapchain properties no longer match.

```c++
uint32_t imageIndex;
VkResult result = vkAcquireNextImageKHR(device, swapchain, UINT64_MAX, imageAvailableSemaphore[currentFrame], VK_NULL_HANDLE, &imageIndex);
if (result == VK_ERROR_OUT_OF_DATE_KHR)
{
    recreateSwapchain();
    continue;
}
else if (result != VK_SUCCESS && result != VK_SUBOPTIMAL_KHR) 
{
    printf("Failed to acquire swapchain image at image index %d", imageIndex);
}
```

If the swapchain is **VK_ERROR_OUT_OF_DATE_KHR** when acquiring an image then it's no longer to present it. If that's the case we recreate the swapchain. If the swapchain **VK_SUBOPTIMAL_KHR** we can still present it so it's considered a success, you can recreate the swapchain if you like.

```c++
result = vkQueuePresentKHR(mainQueue, &presentInfo);
if (result == VK_ERROR_OUT_OF_DATE_KHR || result == VK_SUBOPTIMAL_KHR) 
{
    recreateSwapchain();

    currentFrame = (currentFrame + 1) % maxFramesInFlight;
    continue;
}
else if(result != VK_SUCCESS)
{
    printf("Failed or present swapchain image!");
}
```

This is similar to before but we want to recreate the swapchain if it's **VK_SUBOPTIMAL_KHR** too here to have the best possible result. 

Since we are at the bottom of the render loop here and if you resize the window and hit this case you'll want to advance to the frame here since we have already submitted things to the presentation queue, we should be on the next frame by now.
##### Fixing a deadlock
It's possible to end up resetting the fence too early in the render loop if the **vkAcquireNextImageKHR** as returned **VK_ERROR_OUT_OF_DATE_KHR**. If you reset before this happens the you recreate the swapchain, meaning you didn't submit anything so the fence doesn't get signalled to move past **vkAcquireNextImageKHR**.

So we simply reset the fence after the recreation of the swapchain:
```c++
vkWaitForFences(device, 1, &framesInFlight[currentFrame], VK_TRUE, UINT64_MAX);
        
uint32_t imageIndex;
VkResult result = vkAcquireNextImageKHR(device, swapchain, UINT64_MAX, imageAvailableSemaphore[currentFrame], VK_NULL_HANDLE, &imageIndex);
if (result == VK_ERROR_OUT_OF_DATE_KHR)
{
    recreateSwapchain();
    continue;
}
else if (result != VK_SUCCESS && result != VK_SUBOPTIMAL_KHR) 
{
    printf("Failed to acquire swapchain image at image index %d", imageIndex);
}

vkResetFences(device, 1, &framesInFlight[currentFrame]);
```

Meaning that we definitely reach submission we have submitted something so the fence is signalled OR we return early with a signalled fence still active.
##### VK_ERROR_OUT_OF_DATE isn't always supported
Sometimes the VK_ERROR_OUT_OF_DATE is returned out of the **vkQueuePresentKHR** so in SDL you'll need to make sure a resize event is explicitly. 

```c++
bool running = true;
bool framebufferResize = false;
while (running)
{
    while (SDL_PollEvent(&event))
    {
        switch (event.type)
        {
        case SDL_EVENT_WINDOW_RESIZED:
        {
            framebufferResize = true;
            break;
        }
        }
    }
}
```

Then you'll want to add an extra condition to the present VK_ERROR_OUT_OF_DATE if statement with the bool we added which gets flipped in the **SDL_EVENT_WINDOW_RESIZED** event.

```c++
result = vkQueuePresentKHR(mainQueue, &presentInfo);
if (result == VK_ERROR_OUT_OF_DATE_KHR || result == VK_SUBOPTIMAL_KHR || framebufferResize) 
{
    framebufferResize = false;
    recreateSwapchain();

    currentFrame = (currentFrame + 1) % maxFramesInFlight;
    continue;
}
else if(result != VK_SUCCESS)
{
    printf("Failed or present swapchain image!");
}
```

