2025-10-30 17:17
Status: #baby 
Tags: [[vulkan]] [[render pass]]
# What is a render pass

The render pass is a key part of the graphics pipeline. We need to framebuffer attachments that we will to use while rendering. The render pass handles a couple of things.
- The colour and depth buffers
- How many samples there will be for each buffer.
- How should the buffers and the samples for the rendering operation.

To have all these things you need to attachment description which describes what needs to be changed within a buffer and samples. So a colour attachment description is going to be different to a depth attechment description for example.

You also need a subpass as part of a render pass which describes the operations that are going to be performed when the render pass ends. This also contains the attachment reference which is used to describe the output of fragment shaders which is the layout location and data type. 
# References
##### Main Notes
[[Setting up the colour attachment descriptions]]
[[Setting up the attachment reference]]
[[Setting up the subpass]]
#### Source Notes
[[Vulkan-Tutorial]]