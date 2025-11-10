2025-11-09 15:26
Status: #baby 
Tags: [[vulkan]] [[vulkan swapchain]]
# When to recreate the swapchain image

If you have a function that recreate a swapchain now you just need to know were to apply it. This should be applied when you **vkAcquireNextImageKHR** and when you submit things to be presented with the function **vkQueuePresentKHR**. These both return values you can check for:

1) **VK_ERROR_OUT_OF_DATE_KHR** The swapchain become incompatible with the surface and can no longer be used for rendering. This typical means the window has window resize.
2) **VK_SUBOPTIMAL_KHR** The swapchain can still be used successfully present to the surface, but the swapchain properties no longer match.

```c++
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
```

If the swapchain is **VK_ERROR_OUT_OF_DATE_KHR** when acquiring an image then it's no longer to present it. If that's the case we recreate the swapchain. If the swapchain **VK_SUBOPTIMAL_KHR** we can still present it so it's considered a success, you can recreate the swapchain if you like.

```c++
result = vkQueuePresentKHR(mainQueue, &presentInfo);
if (result == VK_ERROR_OUT_OF_DATE_KHR || result == VK_SUBOPTIMAL_KHR) 
{
    recreateSwapchain();

    currentFrame = (currentFrame + 1) % maxFramesInFlight;
    continue;
}
else if(result != VK_SUCCESS)
{
    printf("Failed or present swapchain image!");
}
```

This is similar to what we did earlier but we want to recreate the swapchain the IF statement takes **VK_SUBOPTIMAL_KHR** condition too because we want the best possible result. 

Since we are at the bottom of the render loop here and if you resize the window and hit this case you'll want to advance to the frame here since we have already submitted things to the presentation queue, we should be on the next frame by now.
# References
##### Main Notes
[[Recreating the swapchain]]
#### Source Notes
[[Drawing a triangle]]