# Reference https://www.youtube.com/watch?v=yeKxsmlvvus

##### What is a subpass
A subpass is part of the render pass that sets up the dependencies and the attachments between the subpasses. 

The subpass actually read and writes to these attachments, this is why you need to set up the dependencies between different subpass so they can read the correct data or write the data need to pass over to the next subpass in the attachment. The reading and writing operations happens at different stages of the graphics pipeline so you need to wait on them too if you were to be the correct dependencies between the subpasses. 

So for example in 1 render pass, you might want to write to a depth buffer with on subpass and another that writes to a colour buffer in a different subpass. These two subpass performance operations at different stages of the graphics pipeline onto different attachments. 

Note these attachments are just image layout attachments, which isn't the same as the attachments for loading and storing data you did when [[Setting up the colour attachment descriptions]] for the render pass itself.
##### Subpass dependencies
Subpasses dependencies do the memory transitions within a render pass automatically depending on the operation and graphics pipeline stage, they will do as soon as they can. Meaning things can happen out of order if you don't set up the dependency.

To use subpasses correctly you pretty much always want to build a subpass dependency chain which tells the order in which subpasses should be executed within a render pass at draw time.

If you don't set up the subpass dependencies on a subpass it will be executed in any order within the render pass, which isn't always an error. It's possible have a subpass that just needs to be executed at some undefined point within a render pass.

You have to be very careful, when you're using a render pass and setting up the sychronisation between the images with semaphore. The operations in the subpass need to happen at the correct time to match up with the semaphores signalling if you don't set up the dependencies you can end up undefined behaviour.

What if you only have 1 subpass within a render pass? Well it's important note that subpasses have self dependencies. Which might need to be set up if you say allowing the vertex shader to run before the swapchain has finished reading the image from the last frame and pausing operation until the swapchain has finish reading the image that we are going to write to later with the colour output. I do this in the {{Setting the sychronisation with the subpass dependencies}}
##### Setting up subpass dependencies
The later brings up subpass dependences which we need to solve. This is done with the **VkSubpassDependency** struct which needs to be set up as part of the render pass creation.

```c++
VkSubpassDependency dependency{};
dependency.srcSubpass = VK_SUBPASS_EXTERNAL;
dependency.dstSubpass = 0;
```

The first two fields specify the indices of the dependency and dependend subpass. 

The indices of the dependencies work so that the **.srcSubpass** takes in a index that is **SMALLER** than the **.dstSupass** (unless **VK_SUBPASS_EXTERNAL** is used). These connect other subpass dependencies together. 

**VK_SUBPASS_EXTERNAL** tells if the dependency starts at the beginning or end of the render pass depending if it's assigned to **.srcSubpass** (which is the beginning) or the **.dstSubpass** (which is the end). You can image a subpass depedency chain being set up like this:

```c++
VkSubpassDependency dependencyOne{};
dependencyOne.srcSubpass = VK_SUBPASS_EXTERNAL;
dependencyOne.dstSubpass = 0;

VkSubpassDependency dependencyTwo{};
dependencyTwo.srcSubpass = 0;
dependencyTwo.dstSubpass = 1;
```

Next we set up what pipeline stages and image attachments we are waiting. When we are using the **.src** paramters we are telling vulkan that **THIS** subpass need to wait on before any operations can begin within that subpass. The **.dst** is all about what needs to be completed before we can pass over this over to the NEXT subpass.

Here we only using 1 subpass within a render pass so these are self dependences. 

```c++
dependency.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
dependency.dstStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
```

For the **.srcStageMask** stage dependencence we are waiting on the swapchain to finish reading the image before we can access it from the previous frame, which is done with the **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** then we can begin writing the final colour of the frame to the swapchain's framebuffer's image.

For **.dstStageMask** the subpass and we need to have completed any work that involves the  **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** which in our case is writing our final colours to the swapchain's framebuffer's image.

```c++
dependency.srcAccessMask = 0;
dependency.dstAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT;
```

For **.srcAccessMask** we don't care what we came in as for our attachments so it's zero. The second one is making sure that subpass attachment has transitioned to a **VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT** before we move on. Meaning we have written to the image.

```c++
renderPassCreateInfo.dependencyCount = 1;
renderPassCreateInfo.pDependencies = &dependency;
```

Then when create the render pass we set up the subpass dependencies for our 1 subpass. 
##### Setting the sychronisation with the subpass dependencies
Without this synchronisation when submitting our command buffer we are assuming that the memory transition goes from undefined to colour attach optimial at the beginning of the render pass, this is WRONG. We haven't acquired the image yet to begin this transition.

This is what this code is set up to do.
```c++
VkSubmitInfo submitInfo{};
submitInfo.sType = VK_STRUCTURE_TYPE_SUBMIT_INFO;

VkSemaphore waitSemaphores[] = { imageAvailableSemaphore };
VkPipelineStageFlags waitStages[] = { VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT };
submitInfo.waitSemaphoreCount = 1;
submitInfo.pWaitSemaphores = waitSemaphores;
submitInfo.pWaitDstStageMask = waitStages;
```

There are actually two way to wait for the image, in the **waitStages[]** array
1) If you set a semaphore the flag is set to **VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT** the ensure that the render passes don't begin until the image is available, avoiding setting up the subpass dependencies. This blocks the pre-colour outputting stages of the graphics pipeline.
2) If you set a semaphore the flag is set to **VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT** which forces the render pass to wait until this part of the graphics pipeline has been finished. However you need to have set up the subpass dependencies within the render pass correctly for this to work, which you should have already done to match the semaphore.