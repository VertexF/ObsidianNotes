2025-11-09 15:41
Status: #baby 
Tags: [[vulkan]] [[vulkan swapchain]]
# Dealing with VK_ERROR_OUT_OF_DATE not being supported

Sometimes the VK_ERROR_OUT_OF_DATE doesn't get returned out of the **vkQueuePresentKHR** so in SDL you'll need to make sure a resize event is explicitly handled. 

```c++
bool running = true;
bool framebufferResize = false;
while (running)
{
    while (SDL_PollEvent(&event))
    {
        switch (event.type)
        {
        case SDL_EVENT_WINDOW_RESIZED:
        {
            framebufferResize = true;
            break;
        }
        }
    }
}
```

Then you'll want to add an extra condition to the **vkQueuePresentKHR** if statement with the bool we added which gets flipped in the **SDL_EVENT_WINDOW_RESIZED** event.

```c++
result = vkQueuePresentKHR(mainQueue, &presentInfo);
if (result == VK_ERROR_OUT_OF_DATE_KHR || result == VK_SUBOPTIMAL_KHR || framebufferResize) 
{
    framebufferResize = false;
    recreateSwapchain();

    currentFrame = (currentFrame + 1) % maxFramesInFlight;
    continue;
}
else if(result != VK_SUCCESS)
{
    printf("Failed or present swapchain image!");
}
```
# References
##### Main Notes
[[Recreating the swapchain]]
[[When to recreate the swapchain image]]
[[Fixing a possible deadlock]]
#### Source Notes
[[Drawing a triangle]]