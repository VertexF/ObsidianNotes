2025-09-26 10:02
Status: #baby 
Tags: [[vulkan]] [[vulkan surface]]
# Creating a vulkan surface

Creating a surface can go two different ways. One you write the platform specific code yourself which might means if you support more than 1 platform. Two you use GLFW or SDL to do the work for you.

It's not that difficult to write the code for Windows you would do something like.

```c++
//get these from SDL/GLFW or the Win32 API
HINSTANCE windowInstance;
HWND window;
VkWin32SurfaceCreateInfoKHR createInfo{};
createInfo.sType = VK_STRUCTURE_TYPE_WIN32_SURFACE_CREATE_INFO_KHR;
createInfo.hwnd = window;
createInfo.hinstance = windowInstance;

VkResult result = vkCreateWin32SurfaceKHR(instance, &createInfo, vulkanAllocatorCallback, &surface);
```

If you want to use SDL/GLFW you would want to do something like. This is what I do.

```c++
glfwCreateWindowSurface(instance, window, nullptr, &surface);
//OR
SDL_Window* window = reinterpret_cast<SDL_Window*>(creation.window);
if (SDL_Vulkan_CreateSurface(window, vulkanInstance, &vulkanWindowSurface) == SDL_FALSE)
{
    aprint("Failed to create Vulkan surface.\n");
}
SDLWindow = window;
```
# References
##### Main Notes
[[Destroying a vulkan surface]]
#### Source Notes
[[Drawing a triangle]]