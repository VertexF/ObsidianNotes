2025-11-07 14:05
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# Acquiring an image from the swapchain

Here we need to get the image from the swapchain. 

```c++
uint32_t imageIndex;
vkAcquireNextImageKHR(device, swapchain, UINT64_MAX, imageAvailableSemaphore[currentFrame], VK_NULL_HANDLE, &imageIndex);
```

The first two parameters are telling vulkan were we get the image from. The third argument is the timeout for this command which we disable again with the UINT64_MAX.

The 4th and 5th argument are to do with waiting for the presentation engine to finish it's work. We are only using a semaphore which will be signalled when the presentation is done. 

The last argument tells us which VkImage is usable in the swapchain array. We use this value to later use to access the correct swapchain image in our render pass as the framebuffer.
# References
##### Main Notes
[[Using a fence to synchronise the previous frame]]
[[Handling frames in flight]]
#### Source Notes
[[Drawing a triangle]]