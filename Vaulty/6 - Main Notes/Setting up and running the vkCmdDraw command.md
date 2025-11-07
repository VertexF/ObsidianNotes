2025-11-05 07:47
Status: #baby 
Tags: [[vulkan]] [[vulkan command buffers]]
# Setting up and running the vkCmdDraw command

This is meant to an introduction to running draw commands since we are using **vkCmdDraw** that's going to draw a triangle. This triangle indices are defined in a vertex shader. This is meant to be the **SIMPLEST** thing you can do within a render pass.

To do any of this you should be currently  [[Recording commands]] and within that you should be [[Using a render pass]] to be able to take the graphics commands and actually draw to something.

After that you'll need to bind the graphics pipeline. 

```c++
vkCmdBindPipeline(commandBuffer, VK_PIPELINE_BIND_POINT_GRAPHICS, mainPipeline);
```

The second argument tells vulkan what type of pipeline you're binding. This is because vulkan needs to know about the attachments in the fragmant shader in the **VkAttachmentReference**

If you've set things up so the viewport and scissor are dynamic when [[Setting up the dynamic states]]
within the graphics pipeline here is were you have to dynamically set up them up.

```c++
VkViewport viewport{};
viewport.x = 0.f;
viewport.y = 0.f;
viewport.width = float(swapchainExtent.width);
viewport.height = float(swapchainExtent.height);
viewport.minDepth = 0.f;
viewport.maxDepth = 1.f;
vkCmdSetViewport(commandBuffer, 0, 1, &viewport);

VkRect2D scissor{};
scissor.offset = { 0, 0 };
scissor.extent = swapchainExtent;
vkCmdSetScissor(commandBuffer, 0, 1, &scissor);
```

This is basically how you set up scissor and viewport statically.

After all that you can simply run the draw command, this is for drawing a triangle. Note it's not index because we have no index buffer.
```c++
vkCmdDraw(commandBuffer, 3, 1, 0, 0);
```

The arguments are after the command buffer:
- **vertexCount** This is for the amount of vertices you want to render. 
- **instanceCount** This is for used for instanced rendering. We have 1 instance so it's 1.
- **firstVertex** This is the offset into a vertex buffer. This defined the lowest value of **gl_VertexIndex**
- **firstInstance** This is used for instanced redering and defines the lowest value of **gl_Instance**

Finally you'll want to end the render pass with
```c++
vkCmdEndRenderPass(commandBuffer);
if (vkEndCommandBuffer(commandBuffer) != VK_SUCCESS) 
{
	printf("Failed to record a command buffer");
}
```
# References
##### Main Notes
[[Setting up the dynamic states]]
[[What is the graphics pipeline]]
[[Using a render pass]]
[[Recording commands]]
#### Source Notes
[[Vulkan-Tutorial]]