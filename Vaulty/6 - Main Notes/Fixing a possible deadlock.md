2025-11-09 15:32
Status: #baby 
Tags: [[vulkan]] [[vulkan swapchain]]
# Fixing a possible deadlock

It's possible to end up resetting the fence too early in the render loop if the **vkAcquireNextImageKHR** has returned **VK_ERROR_OUT_OF_DATE_KHR**. If you **vkResetFences** and result the render loop, **WITHOUT** submitting anything the fence will look the CPU because the fence didn't get signalled

So we simply reset the fence **AFTER** the recreation of the swapchain
```c++
vkWaitForFences(device, 1, &framesInFlight[currentFrame], VK_TRUE, UINT64_MAX);
        
uint32_t imageIndex;
VkResult result = vkAcquireNextImageKHR(device, swapchain, UINT64_MAX, imageAvailableSemaphore[currentFrame], VK_NULL_HANDLE, &imageIndex);
if (result == VK_ERROR_OUT_OF_DATE_KHR)
{
    recreateSwapchain();
    continue;
}
else if (result != VK_SUCCESS && result != VK_SUBOPTIMAL_KHR) 
{
    printf("Failed to acquire swapchain image at image index %d", imageIndex);
}

vkResetFences(device, 1, &framesInFlight[currentFrame]);
```

This avoids the deadlock because we'll definitely reach submission and the fence will be signalled OR we return early with a signalled fence still active.
# References
##### Main Notes
[[When to recreate the swapchain image]]
[[Recreating the swapchain]]
#### Source Notes
[[Drawing a triangle]]