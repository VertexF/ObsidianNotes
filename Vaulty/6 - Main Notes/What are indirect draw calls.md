2026-02-13 07:53
Status: #baby 
Tags: [[vulkan]] [[indirect draw calls]]
# What are indirect draw calls

Indirect draw calls use a draw command buffer this is a buffer that gets written to in a shader and read out again in a shader. This is known as GPU driven rendering.

The idea with GPU driven rendering and indirect draw calls is to offload the draw call generation to the GPU. This allows you to cull geometry on the GPU, which can then reduce the amount of draw calls because we are allowing the GPU to generate draw calls.
# References
##### Main Notes
[[Advantages of indirect draw calls]]
#### Source Notes
[[GPU Rendering and Multi-Draw Indirect]]