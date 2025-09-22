# Reference https://www.youtube.com/watch?v=e14z9oOsPu0

Quick overview of the handles that are part of the fundamental part of setting a Vulkan application. Considered these the minimum.
- VkInstance is the instance were everything you do with vulkan is based upon.
- VkSurfaceKHR is the window surface that's the window to your OS.
- VkPhysicalDevice this is the chosen physical device vulkan is going to do work on.
- VkQueue is a queue that sends messages to the GPU/VkPhysicalDevice.
- VkDevice this allows layer of abstraction to the physical device, this allows us to configure how we want to interact the hardware.
- VkSwapchainKHR This sends VkImages to the monitor and gives up VkImages to renderer to with our shaders.


