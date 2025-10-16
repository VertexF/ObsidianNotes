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

#### Checking swap chain support
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