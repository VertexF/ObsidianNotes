2025-11-21 09:41
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# What are descriptors

These guys are used to update shaders every frame with global variables. These can be buffers or images. An example usage would be providing the model view projection matrix in a buffer to update the vertex shader every frame. 

To use these you need to set up these three things:
1) You need to specify a [[What is a descriptor set layout|descriptor set layout]] during pipeline creation in the stage of the pipeline layout.
2) You need to allocate descriptors like command buffer from a memory pool, called a descriptor pool.
3) Bind the descriptor you want to use during rendering.
# References
##### Main Notes
[[Setting up the pipeline layout with a descriptor]]
[[What is a descriptor set layout]]
#### Source Notes
[[Uniform buffers]]