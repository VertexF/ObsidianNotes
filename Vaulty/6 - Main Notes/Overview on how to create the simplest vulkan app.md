2025-09-26 12:27
Status: #baby 
Tags: [[vulkan]]
# Overview on how to create the simplest vulkan app

##### **NOTE**: This is imcomplete and needs a rewrite

First you need to create the instance and step up the instance level extensions which are on Windows. Check that these are available for Windows. 

```c++
	const char* REQUESTED_EXTENSIONS[] =
    {
        VK_KHR_SURFACE_EXTENSION_NAME,
        VK_KHR_WIN32_SURFACE_EXTENSION_NAME,
        VK_EXT_DEBUG_REPORT_EXTENSION_NAME,
        VK_EXT_DEBUG_UTILS_EXTENSION_NAME
    };
```

Then you want to create the validation layer, with the debug util call back function. Make sure you have the vulkan SDK installed so you have access to the LunarG validation. Once you've done that you, load in the, create/delete function, then you create the debug util function and link it to the instance creation.

Then you want to create the surface for the given OS your one. Select a GPU with the physical device creation that has the queue families, features and properties you want. After that you want to list the queue families from the GPU and keep track of the indices that include presentation and graphic drawing command abilities.

Next is about logical device creation. You want to check for the device level extensions which should at least include the swapchain extension and any features you want to use out of the available bunch you queried when looking at physical device selection. You may want to turn on validation layers for device level things to support old GPUs. You also need to create the queue which is were the command buffers will be created from. Then create the device. 
# References
##### Main Notes
[[What is Vulkan]]
[[First Step for setting up Vulkan]]
[[What are validation layers]]
[[What is a vulkan surface]]
[[What is a vulkan physical device]]
[[What are queue families]]
[[What is vulkan logical device]]

#### Source Notes
