2025-12-06 19:18
Status: #baby 
Tags: [[vulkan]] [[depth testing]]
# Setting depth buffer attachments

This is part of the graphics pipeline is to do with testing if pixels are in front of another pixel. When setting up the graphics pipeline you can set this to null skipping this step.

This is extremely familar to [[Setting up the colour attachment descriptions]] but we do it for depth instead. So lets get started

```c++
VkAttachmentDescription depthAttachment{};
depthAttachment.format = findDepthFormat();
depthAttachment.samples = VK_SAMPLE_COUNT_1_BIT;
depthAttachment.loadOp = VK_ATTACHMENT_LOAD_OP_CLEAR;
depthAttachment.storeOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
depthAttachment.stencilLoadOp = VK_ATTACHMENT_LOAD_OP_DONT_CARE;
depthAttachment.stencilStoreOp = VK_ATTACHMENT_STORE_OP_DONT_CARE;
depthAttachment.initialLayout = VK_IMAGE_LAYOUT_UNDEFINED;
depthAttachment.finalLayout = VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL;
```

The format is the **depth image itself**. Unlike the colour attachment description we don't care about **.storeOp** as it will not be used faster drawing has finished. Everything else explains itself.

This is extactly the same as [[Setting up the attachment reference]] and [[Setting up the subpass]] we just add depth now.
```c++
VkAttachmentReference depthAttachmentRef{};
depthAttachmentRef.attachment = 1;
depthAttachmentRef.layout = VK_IMAGE_LAYOUT_DEPTH_STENCIL_ATTACHMENT_OPTIMAL;

VkSubpassDescription subpass{};
subpass.pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS;
subpass.colorAttachmentCount = 1;
subpass.pColorAttachments = &colourAttachmentRef;
subpass.pDepthStencilAttachment = &depthAttachmentRef;
```

Unlike the colour attachment, a subpass can only use 1 depth stencil attachment. As it doesn't really make sense to do depth test on multiple buffers.

```c++
VkSubpassDependency dependency{};
dependency.srcSubpass = VK_SUBPASS_EXTERNAL;
dependency.dstSubpass = 0;
dependency.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT | VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT;
dependency.srcAccessMask = VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT;
dependency.dstStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT | VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT;
dependency.dstAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT | VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT;
```

Finally we need to extend the subpass dependencies to make sure there is no conflict between the transitioning of the depth image and it being cleared as of its load operation. We first access the depth buffer **VK_PIPELINE_STAGE_EARLY_FRAGMENT_TESTS_BIT** stage which is were the load operation clears things. We need to specify the access mask to write with **VK_ACCESS_DEPTH_STENCIL_ATTACHMENT_WRITE_BIT**, after the **VK_PIPELINE_STAGE_LATE_FRAGMENT_TESTS_BIT**.
# References
##### Main Notes
[[Setting up the colour attachment descriptions]]
[[Setting up the attachment reference]]
[[Setting up the subpass]]
#### Source Notes
[[Drawing a triangle]]
[[Depth buffering]]
