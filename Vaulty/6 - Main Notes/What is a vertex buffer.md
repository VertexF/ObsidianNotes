2025-11-16 14:33
Status: #baby  
Tags: [[vulkan]] [[vulkan buffer]]
# What is a vertex buffer

These can store vertex attributes that are read by the GPU. They also don't allocate there own memory, meaning you'll have to do extra work to load in the data needed.

This is one way to load things into shaders you can use other ways. It requires set up on the graphics pipeline to use vertex buffers. 
# References
##### Main Notes
[[Shader vertex input descriptions]]
[[Setting up binding descriptions]]
[[Setting up the vertex attributes descriptions]]
[[Setting up the a buffer]]
#### Source Notes
[[Vertex buffer]]