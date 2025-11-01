2025-11-01 08:50
Status: #baby 
Tags: [[vulkan]] [[render pass]]
# Setting up the attachment reference

Every subpass need an **VkAttachmentReference** which describes the output of the shader in it's location and format. This describes the output of the fragment shader, so when you write 

```glsl
layout(location = 0) out vec4 outColour
```

This is described here in vulkan with 

```c++
VkAttachmentReference colourAttachmentRef{};
colourAttachmentRef.attachment = 0;
colourAttachmentRef.layout = VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL;
```

The **.layout** is the format of the output of the shader so this is an attachment optimal for the colour output, this will be different if your output is different. The **.attachment** is the `layout(location = 0)` number found in your shader.
# References
##### Main Notes
[[Setting up the subpass]]
[[What is a render pass]]
#### Source Notes
[[Vulkan-Tutorial]]