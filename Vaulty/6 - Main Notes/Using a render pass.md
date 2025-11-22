2025-11-04 19:38
Status: #baby 
Tags: [[vulkan]] [[render pass]] [[vulkan command buffers]]
# Using a render pass

To use a render pass you need to have finished [[Setting up the render pass]] and began [[Recording commands]] as using a render pass is part of the graphics commands.

To start drawing you'll need to begin the render pass using the **vkCmdBeginRenderPass** using the **VkRenderPassBeginInfo** struct. 

```c++
VkRenderPassBeginInfo renderPassBeginInfo{};
renderPassBeginInfo.sType = VK_STRUCTURE_TYPE_RENDER_PASS_BEGIN_INFO;
renderPassBeginInfo.renderPass = renderPass;
renderPassBeginInfo.framebuffer = swapchainFramebuffers[imageIndex];
```

**.renderPass** is the render pass that you should have set up as part of the graphics pipeline.

**.framebuffer** is the VkImageView you're going to write to when you run graphics commands. Here we are writing straight to the swapchain images in this example, but you can write to another VkImageView if you like.

We aquire the **imageIndex** from the swapchain before we begin the render pass to give up a safe image to write to.

Then we have to tell vulkan the areas were the load operations and and the store operation read and write to which you set up with the attachment description in the render pass.

```c++
renderPassBeginInfo.renderArea.offset = { 0, 0 };
renderPassBeginInfo.renderArea.extent = swapchainExtent;
```

Since we are drawing straight onto a swapchain image extents, pixels outside this range are undefined. You want your attachment description bounds to be the same as the **.renderArea** for better performance.

```c++
VkClearValue clearColour = { { { 0.218f, 0.f, 0.265f, 1.f } } };
renderPassBeginInfo.clearValueCount = 1;
renderPassBeginInfo.pClearValues = &clearColour;
```

Remember when we set the **.loadOp** in the VkAttachmnetDescription to be **VK_ATTACHMENT_LOAD_OP_CLEAR** for clearing the screen when were [[Setting up the colour attachment descriptions]]? This is what things will be cleared back to. It's a good idea not to use black as it's easier to tell if something is working or not. Here we are not clearing depth and stencil but you can do that to.

With all that we can begin the render pass, record draw commands and end the render pass.
```c++
vkCmdBeginRenderPass(commandBuffers[currentFrame], &renderPassBeginInfo, VK_SUBPASS_CONTENTS_INLINE);

//... Draw commands here

vkCmdEndRenderPass(commandBuffers[currentFrame]);
```

As you can see in the comments that's where all the draw commands. The last parameter in the **vkCmdBeginRenderPass** can be two different values:
- **VK_SUBPASS_CONTENTS_INLINE** The commands will be embedded in the primary command buffer, there will be no secondary command buffer. 
- **VK_SUBPASS_CONTENTS_SECONDARY_COMMAND_BUFFERS** The render pass be will executed from a secondary command buffer.
  
Note that these values can change for each render pass you are using. If you're for example calling out to subpass in one render pass then  doing everything inline. 
# References
##### Main Notes
[[Setting up and running the vkCmdDraw command]]
[[Recording commands]]
[[Setting up the render pass]]
[[Setting up the colour attachment descriptions]]
[[Handling frames in flight]]
#### Source Notes
[[Drawing a triangle]]