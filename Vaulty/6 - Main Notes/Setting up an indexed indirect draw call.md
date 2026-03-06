2026-02-13 10:33
Status: #baby 
Tags: [[vulkan]] [[indirect draw calls]]
# Setting up an indexed indirect draw call

When you want to use an indirect draw call, you first need to use the struct **VkDrawIndexedIndirectCommand**, this provide the information of the vertices and indices to draw.

Since you can offset into an index buffer and vertex buffer using the **VkDrawIndexedIndirectCommand** struct, you can make a HUGE index buffer that contains a bunch of models in a scene bind that once, even if the total number of vertices exceed 2^16 or 2^32 in that draw call. This can **ONLY WORK IF** each portion of the index buffer is zero-indexing. Meaning that if each model has say an index buffer of (0, 5, 6, ....) and (99, 0, 120, ...) they can live in one index buffer.

Okay this is important, you don't create the **VkDrawIndexedIndirectCommand** on the CPU but on the GPU, that's the whole point of this. It gets filled on a compute shader then you pass up the **VkDrawIndexedIndirectCommand** in an indirect draw call which is read only and generates the draw calls for indirect index draw call.
# References
##### Main Notes
[[How to use an indexed indirect draw call]]
[[Advantages of indirect draw calls]]
#### Source Notes
[[GPU Rendering and Multi-Draw Indirect]]
