2025-10-30 19:21
Status: #baby 
Tags: [[vulkan]] [[render pass]]
# Setting up the subpass

A subpass sub divides render passes into seperate logical phases. The benefit of using multiple subpasses is that the GPU can do some optimisations rather than adding a tonne of render passes.

A subpass is a collection of render operation that happen in sequence and that also depend on the framebuffer previous frame/pass. For example sequence of post-processing events can happen within a subpass.

Once you've finished [[Setting up the attachment reference]] you can create the subpass with the **VkSubpassDescription** 

```c++
VkSubpassDescription subpass{};
subpass.pipelineBindPoint = VK_PIPELINE_BIND_POINT_GRAPHICS;
```

There is a compute one but it's currently not supported but will be in the future so graphics it is.

Then you want to tell **VkSubpassDescription** the amount of attachment references you have. If you have more than 1 then this will be in an array.

```c++
subpass.colorAttachmentCount = 1;
subpass.pColorAttachments = &colourAttachmentRef;
```

There are 4 other attachments for different shaders:
- **.pInputAttachment** This is for reading things out of a shader
- **.pResolveAttachment** This is for muiltisampling colour attachments
- **.pDepthStencilAttachment** This for depth and stencil data.
- **.pPreserveAttachments** This is for things that are used by the subpass but we need to keep that data around.
# References
##### Main Notes
[[Setting up the attachment reference]]
[[What is a render pass]]
#### Source Notes
[[Drawing a triangle]]