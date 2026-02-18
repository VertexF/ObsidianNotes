2026-02-13 10:28
Status: #baby 
Tags: [[vulkan]] [[draw calls]]
# Advantages of indirect draw calls

A common idea when it comes to rendering, you iterate through each model to draw in a scene and bind there resource such as vertex buffers, index buffer and descriptors before the draw call. One thing to note is that this isn't free in command buffer generation such as **vkCmdBindVertexBuffer** and rendering.

You could use indirect draw calls instead introduces in Vulkan 1.2 which queries and GPU buffer to get it's draw paramters.

You have two advantages here:
1) Draw calls can be generating on the GPU such as in compute shader (which can do additional work to cull things.)
2) Using an array of draw calls can be called once, reducing the command buffer overhead.
# References
##### Main Notes
[[What are indirect draw calls]]
#### Source Notes
[[GPU Rendering and Multi-Draw Indirect]]