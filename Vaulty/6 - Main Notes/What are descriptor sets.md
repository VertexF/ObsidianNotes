2025-11-21 09:57
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# What are descriptor sets

The descriptor set specifies the actual buffer or image that will be bound to descriptors. The descriptor set is similar to a framebuffer in some ways, as the framebuffer specifies the the VkImageView that will be bound to the render pass. Like a render pass instance needs a framebuffers VkImageView, we need the descriptor set to carry the actually buffer to the shader.

There are different types of descriptor sets such SSBO (Shader Storage Buffer Object) and UBO (Uniform Buffer Object).
# References
##### Main Notes
[[What are descriptors]]
[[What is a descriptor set layout]]
#### Source Notes
[[Uniform buffers]]