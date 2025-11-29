2025-11-07 14:08
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# Submitting the command buffer for rendering

Queue submission and synchronisation is configured with **VkSubmitInfo**. 

```c++
VkSubmitInfo submitInfo{};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;

VkSemaphore waitSemaphores[] = { imageAvailableSemaphore[currentFrame] };
VkPipelineStageFlags waitStages[] = { VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT };
submitInfo.waitSemaphoreCount = 1;
submitInfo.pWaitSemaphores = waitSemaphores;
submitInfo.pWaitDstStageMask = waitStages;
```

What we are doing here is setting up part of the pipeline stages that need to wait. We need to wait on the **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** This is the part of that handles the output of the final colour. This stop things writing to an image until it is ready.

This is further explained here [[Setting the synchronisation with the subpass dependencies]] 

This means that we can run the vertex shader and other stages of the pipeline before we have to wait. We explain why we are doing this in the subpass dependency section.

```c++
submitInfo.commandBufferCount = 1;
submitInfo.pCommandBuffers = &commandBuffers[currentFrame];
```

Here we submit the command buffer for execution.

```c++
VkSemaphore signalSemaphores[] = { renderFinishSemaphore[currentFrame] };
submitInfo.signalSemaphoreCount = 1;
submitInfo.pSignalSemaphores = signalSemaphores;
```

This is simply the set up to signal the semaphore when the command buffer has finished executing.

```c++
if (vkQueueSubmit(mainQueue, 1, &submitInfo, framesInFlight[currentFrame]) != VK_SUCCESS) 
{
	printf("Failed to submit the command buffer to draw with the current queue.");
}
```

For this function we have
- **mainQueue** which is queue we are submitting the command buffer on.
- **1** which is the total count of **VkSubmitInfo** in an array.
- **&submitInfo** is the struct itself or an array of them. Having an array can help with efficiency if the work load is large enough.
- **frameInFlight** is our fence this will signal the fence to wait for the command buffer to finish executing. This is so our frames don't over draw each other causing all sort of issues.
# References
##### Main Notes
[[What are command buffers]]
[[What are semaphores]]
[[What are fences]]
[[Acquiring an image from the swapchain]]
#### Source Notes
[[Drawing a triangle]]
