2025-11-07 15:43
Status: #baby 
Tags: [[vulkan]] [[vulkan subpass]] [[vulkan sychronisation]]
# Setting the synchronisation with the subpass dependencies

You should have finished [[Setting up subpass dependencies with one subpass]] which will allow our subpass to behavour properly within our render pass instance. 

To actually get the dependency within the subpass to work correctly this how you should be setting things up:
```c++
VkSubmitInfo submitInfo{};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;

VkSemaphore waitSemaphores[] = { imageAvailableSemaphore[currentFrame]};
VkPipelineStageFlags waitStages[] = { VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT };
submitInfo.waitSemaphoreCount = 1;
submitInfo.pWaitSemaphores = waitSemaphores;
submitInfo.pWaitDstStageMask = waitStages;
```

The wait stage is at **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** which is the same point we are waiting in our subpass dependency.

There are actually two way to wait for the image, in the **waitStages[]** array
1) If you set a semaphore the flag is set to **VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT** this ensure that the render passes don't begin until the image is available, avoiding setting up the subpass dependencies. This blocks the pre-colour outputting stages of the graphics pipeline.
2) If you set a semaphore the flag is set to **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** which forces the render pass to wait until this part of the graphics pipeline has been finished. However you need to have set up the subpass dependencies within the render pass correctly for this to work, which you should have already done to match the semaphore.
# References
##### Main Notes
[[What is a subpass]]
[[What are subpass dependencies]]
[[Setting up subpass dependencies with one subpass]]
#### Source Notes
[[Drawing a triangle]]
[[Render Passes in Vulkan]]