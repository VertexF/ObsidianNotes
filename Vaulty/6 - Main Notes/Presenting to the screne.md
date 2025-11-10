2025-11-08 09:53
Status: #baby 
Tags: [[vulkan]]
# Presenting to the screne

If you've done everything needed to take the image to the submit your command buffer, the final to is tell the swapchain to submit your image to the screen. This is done with the **VkPresentInfoKHR** struct.

This code should all go at the end of the drawing a frame.

```c++
VkPresentInfoKHR presentInfo{};
presentInfo.sType = VK_STRUCTURE_TYPE_PRESENT_INFO_KHR;
presentInfo.waitSemaphoreCount = 1;
presentInfo.pWaitSemaphores = signalSemaphores;
```

Here we are waiting on the signalSemaphore which was used when we submitted the command buffer. Meaning we are waiting for everything to render and write that result out.

```c++
VkSwapchainKHR swapchains[] = { swapchain };
presentInfo.swapchainCount = 1;
presentInfo.pSwapchains = swapchains;
presentInfo.pImageIndices = &imageIndex;
presentInfo.pResults = nullptr;
```

Here we actually present the specify swapchain with **.pSwapchains** to present the images. The index of the image also needs to be presented with **.pImageIndices**. **.pResults** takes an array of VkResults which is useful if your using multiple swapchains because it will tell you if the images were presented successfully.

```c++
vkQueuePresentKHR(mainQueue, &presentInfo);
```

All this function is tell the swapchain to present the image to the screen. 

When closing you need to remember that GPU is asynchronous from the CPU meaning that operations on the GPU might be still happening so you'll need to add

```c++
vkDeviceWaitIdle(device);
```

all to pause operations. This is a very crewd way of performaning synchronisations between the GPU and CPU but while closing this is enough.
# References
##### Main Notes

#### Source Notes
[[Drawing a triangle]]