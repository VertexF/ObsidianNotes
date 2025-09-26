2025-09-26 09:40
Status: #baby 
Tags: [[vulkan]] [[vulkan surface]]
# What is a vulkan surface

A surface is an abstraction that allows vulkan to support the swapchain in presenting images to the screen. 

Vulkan is platform agonistic, so it needs to be extended to be able support a given OS by getting vulkan to use the OS Window System Integration (WSI).

For example, if you're on Windows you'll want a window surface, which is an extension which allows us to present rendered images to a screen on Windows.

As it's an extension you it's completely optional if you just want to do offscreen rendering. You still create the swapchain but have no surface for it to do anything with images.
# References
##### Main Notes
[[Vulkan surface extensions]]
#### Source Notes
[[Vulkan-Tutorial]]
