2025-11-21 09:44
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# What is a descriptor set layout

Descriptor set layouts tells the graphics pipeline what resources are going to be accessed by the graphics pipeline as global variables. These are similar to render passes in the sense as a render pass tells us the type of attachments that are going to be used, when setting up the graphics pipeline. We do the same but with buffers specify the shader it's going to be in.

These handle things like binding and what stage will the descriptors will be used in.  
# References
##### Main Notes
[[What are descriptors]]
[[Creating a descriptor set layout]]
#### Source Notes
[[Uniform buffers]]