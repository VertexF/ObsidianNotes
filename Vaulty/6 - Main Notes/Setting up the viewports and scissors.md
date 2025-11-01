2025-10-25 17:17
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# Setting up the viewports and scissors

The viewport decides what part of the framebuffer is outputted for rendering. For a standard set up you want the entire thing.

```c++
VkViewport viewport{};
viewport.x = 0.f;
viewport.y = 0.f;
viewport.width = float(swapchainExtent.width);
viewport.height = float(swapchainExtent.height);
viewport.minDepth = 0.f;
viewport.maxDepth = 1.f;
```

Remeber that that swapchain images (VkImage) can be larger than the application window size. That's why we are using the swapchain extents.

If you're using a standard set up you'll want min and max depth to be 0.f to 1.f. If you're doing reverse perspective you'll want the opposite.

The viewport defines the transformation from image to the framebuffer, but the scissort defines the rectangle that's actually stored. Meaning pixels outside the scissor will be discards.

![[viewports_scissors-2294964527.png]]

You can see the scissor cuts the rendered image in half, however the viewport scales the image depending on the size. If you want the entire screen to be displayed set things up like this

```c++
VkRect2D scissor{};
scissor.offset = { 0, 0 };
scissor.extent = swapchainExtent;
```

If you want to set things up statically with a viewport and scissor that doesn't change you'll want to set things up like this, without the comments. Otherwise do it with the comments along with [[Setting up the dynamic states]]

```c++
VkPipelineViewportStateCreateInfo viewportCreateInfo{};
viewportCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_VIEWPORT_STATE_CREATE_INFO;
viewportCreateInfo.viewportCount = 1;
//viewportCreateInfo.pViewports = &viewports;
viewportCreateInfo.scissorCount = 1;
//viewportCreateInfo.pScissors = &scissors;
```

# References
##### Main Notes
[[Setting up the dynamic states]]
What is a framebuffer <- This needs creating for this note.
#### Source Notes
[[Vulkan-Tutorial]]