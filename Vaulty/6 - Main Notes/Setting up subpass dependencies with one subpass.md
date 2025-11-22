2025-11-07 15:32
Status: #baby 
Tags: [[vulkan]] [[vulkan subpass]]
# Setting up subpass dependencies with one subpass

To setup the subpass self dependency for 1 subpass we use the **VkSubpassDependency** struct which needs to be set up as part of the render pass creation. When and where you want to pause needs to be thought about at draw time when the subpass is used. We are trying to synchronising rendering with the swapchain with this self dependency

```c++
VkSubpassDependency dependency{};
dependency.srcSubpass = VK_SUBPASS_EXTERNAL;
dependency.dstSubpass = 0;
```

The first two fields specify the indices of the dependency graph. 

The indices of the dependencies work so that the **.srcSubpass** takes in a index that is **SMALLER** than the **.dstSupass** (unless **VK_SUBPASS_EXTERNAL** is used). These connect other subpass dependencies together. 

**VK_SUBPASS_EXTERNAL** tells vulkan the dependency starts at the beginning or end of the render pass depending if it's assigned to **.srcSubpass** (which is the beginning) or the **.dstSubpass** (which is the end). You can imagine a subpass depedency chain being set up like this:

```c++
VkSubpassDependency dependencyOne{};
dependencyOne.srcSubpass = VK_SUBPASS_EXTERNAL;
dependencyOne.dstSubpass = 0;

VkSubpassDependency dependencyTwo{};
dependencyTwo.srcSubpass = 0;
dependencyTwo.dstSubpass = 1;
```

Next we set up what pipeline stages and image attachments we are waiting. When we are using the **.src** paramters we are telling vulkan that **THIS** subpass needs to wait on before any operations can begin within that subpass. The **.dst** is all about what needs to be completed before we can pass over this over to the NEXT subpass.

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

For **.srcAccessMask** we don't care what access attachment we came in with so we set that to zero. The second one is making sure that subpass attachment has transitioned to a **VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT** before we move on. Meaning we have written to the image.

```c++
renderPassCreateInfo.dependencyCount = 1;
renderPassCreateInfo.pDependencies = &dependency;
```

Then when create the render pass we set up the subpass dependencies for our 1 subpass.
# References
##### Main Notes
[[What is a subpass]]
[[What are subpass dependencies]]
#### Source Notes
[[Drawing a triangle]]
[[Render Passes in Vulkan]]