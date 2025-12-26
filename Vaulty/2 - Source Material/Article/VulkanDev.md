# Reference https://vkguide.dev/docs/introduction

GPU driven rendering is when you use a compute shader to handle rendering to be able to render hundreds of thousands of meshes.

A good idea for graphics pipeline creation is to create them on a background thread.

##### GPU features 
In the vulkandev tutorial it talks about 4 GPU features that are used.

```c++
VkPhysicalDeviceVulkan12Features vulkan12Features{};
vulkan12Features.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_2_FEATURES;
vulkan12Features.pNext = nullptr;
vulkan12Features.bufferDeviceAddress = VK_TRUE;
vulkan12Features.descriptorIndexing = VK_TRUE;
void* currentPNext = nullptr;

VkPhysicalDeviceVulkan13Features vulkan13Features{};
vulkan13Features.sType = VK_STRUCTURE_TYPE_PHYSICAL_DEVICE_VULKAN_1_3_FEATURES;
vulkan13Features.pNext = &vulkan12Features;
vulkan13Features.dynamicRendering = VK_TRUE;
vulkan13Features.synchronization2 = VK_TRUE;
currentPNext = &vulkan13Features;
```

These features allow things to be done in an easier way or more performant if they're available.
- **.dynamicRendering** completely skips the render pass and framebuffer set up part of vulkan.
- **.synchronization2** this updates the synchronisation functions.
- **.bufferDeviceAddress** allows you to have a pointer to the `VkBuffer` in your shaders, which is one half of going bindless.
- **.descriptorIndexing** with this you treat descriptor memory as one massive array, and we can freely access any resource we want at any time, by indexing, which is the over half of going bindless.

**VK_PRESENT_MODE_FIFO_RELAXED** can be explained by it's result. If you have a 60Hz monitor and you're application is running at 55Hz, the presention engine will drop the screen presentation to an interval that's divisible by the monitor refresh rate. In this case it will be 30Hz.
##### Command buffers and pools
To be able to use command buffer in different threads, you'll need a command buffer and command pool per-thread and make sure that that each thread only uses the command buffer and pool they created. If you do that, then you can submit either command buffer is any thread.

Saying that **vkQueueSubmit** is not thread-safe. You can only have 1 thread push the command on a given queue at a time. It's common for a background thread to have things submitted and wait on this submission which is pretty expensive, while the main render thread goes on and continues executing. 
###### Command buffer states
When you first create a command buffer it starts in a **Ready** state. Once you run a **vkBeginCommandBuffer** you put the command buffer in a **Record** state. Once you run **vkEndCommandBuffer** the command buffer is an **Executable** state. 

After submitting a command buffer it does into a **Pending** state. It's in a pending state because you need to wait for the work to be done then you reset it, with **vkResetCommandBuffer**. 

This is why I've set things up that for each frame we are going to use within a swapchain, we have command buffers for each image. 