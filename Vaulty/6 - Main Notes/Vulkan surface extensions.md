2025-09-26 09:54
Status: #baby 
Tags: [[vulkan]] [[vulkan surface]]
# Vulkan surface extensions

When you're rendering you need at least 2 extensions. One for accessing the **VkSurfaceKHR** which is a platform agnostic surface. Two the platform specific extension.  

These are for Windows **VK_KHR_surface** with the macro **VK_KHR_SURFACE_EXTENSION_NAME** and **VK_KHR_win32_surface** with the macro **VK_KHR_WIN32_SURFACE_EXTENSION_NAME**.

These are the extensions I would consider foundational to rendering on Windows.

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
# References
##### Main Notes
[[What is a vulkan surface]]
[[Checking for supported extensions]]
#### Source Notes
[[Vulkan-Tutorial]]
