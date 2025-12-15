2025-11-16 16:59
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Binding the vertex buffer and drawing

To use the vertex buffer you've gotta record the binding.

```c++
VkBuffer vertexBuffers[] = { vertexBuffer };
VkDeviceSize offsets[] = { 0 };
vkCmdBindVertexBuffers(commandBuffers[currentFrame], 0, 1, vertexBuffers, offsets);

vkCmdDraw(commandBuffers[currentFrame], uint32_t(vertices.size()), 1, 0, 0);
```

The second two parameters in the **vkCmdBindVertexBuffers** function are the firstBinding which is the index of the first vertex input binding and bindingCount which is how many vertex buffers you're binding.

We also need to change the **vkCmdDraw** to actually set the vertices size which is how many vertices we have.
# References
##### Main Notes
[[Setting up and running the vkCmdDraw command]]
[[Setting up the a buffer]]
[[Filling up a vertex buffer with CPU visibility]]
#### Source Notes
[[Vertex buffer]]