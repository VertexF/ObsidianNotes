2025-09-21 08:56
Status: #adult 
Tags: [[vulkan]]
# Dealing with VK_EXTENSION_NOT_PRESENT error

If you see this error you haven't correctly set up the extension you are trying to use or it's not supported on the graphics cards you selected. First make sure it's supported [[Checking for supported extensions]] If it is supported you will need to turn it. 

Here we are using the common example of setting up a custom callback function.

To set this up you need to add the list of extensions you already have **VK_EXT_DEBUG_REPORT_EXTENSION_NAME** If you don't do this you will get the **VK_EXTENSION_NOT_PRESENT** when setting things up.

These should be the default extensions **ONLY** in debug. Remove the debug extensions on release builds of your application. Anything else you need just add it to the list assuming it's supported.
```c++
	const char* REQUESTED_EXTENSIONS[] =
    {
        VK_KHR_SURFACE_EXTENSION_NAME,
        VK_KHR_WIN32_SURFACE_EXTENSION_NAME,
        VK_EXT_DEBUG_REPORT_EXTENSION_NAME,
        VK_EXT_DEBUG_UTILS_EXTENSION_NAME
    };
```

# References
##### Main Notes
[[Setting up extensions]]
[[Checking for supported extensions]]
[[Why set up debug callbacks for vulkan validation]]
[[Creating the custom callback function for validation]]
#### Source Notes
[[Drawing a triangle]]