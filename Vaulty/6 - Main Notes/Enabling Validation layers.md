2025-09-21 08:17
Status: #adult 
Tags: [[vulkan]] [[vulkan validation]]
# Enabling Validation layers

To enable layers you need to simply wire them up with a const char* array and the count. To have access to the validation layer you need to install the LunarG SDK and use the string **"VK_LAYER_KHRONOS_validation"** in your const char* array. 

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
#if defined(VULKAN_DEBUG_REPORT)
        createInfo.enabledLayerCount = ArraySize(REQUESTED_LAYERS);
        createInfo.ppEnabledLayerNames = REQUESTED_LAYERS;
#else
        createInfo.enabledLayerCount = 0;
        createInfo.ppEnabledLayerNames = nullptr;
#endif //VULKAN_DEBUG_REPORT
```

You will want to have a "#define" or use NDEBUG and DEBUG to enable and disable these layers when compiling different versions of your application. That's what I've done above here.
# References
##### Main Notes
[[Vulkan Layers aren't extensions]]
[[Instance and device level validation]]
#### Source Notes
[[Drawing a triangle]]