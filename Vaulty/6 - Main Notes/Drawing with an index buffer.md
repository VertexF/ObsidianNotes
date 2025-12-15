2025-11-19 19:30
Status: #baby 
Tags: [[vulkan]] [[vulkan buffer]]
# Drawing with an index buffer

First thing to note when using a index buffer you **CAN'T** bind more than one at a time. You **CAN'T** use different index buffer for different vertex attributes, if you want to do that you'll need to duplicate the vertex data even if one attribute is different.

Knowing that you'll want to bind our index buffer

```c++
vkCmdBindVertexBuffers(commandBuffers[currentFrame], 0, 1, vertexBuffers, offsets);
vkCmdBindIndexBuffer(commandBuffers[currentFrame], indexBuffer, 0, VK_INDEX_TYPE_UINT16
```

The arguments go
1) The command buffer
2) The index buffer
3) The byte offset into the index buffer
4) The data type which can either VK_INDEX_TYPE_UINT16 or VK_INDEX_TYPE_UINT32

Next we need a new draw call to use the index buffer
```c++
vkCmdDrawIndexed(commandBuffers[currentFrame], uint32_t(indices.size()), 1, 0, 0, 0);
```

The arguments go
1) The command buffer
2) The number of indices
3) The number of instances (if you're not instance rendering set this to 1)
4) The offset into the index buffer (if you start at 1 for example the first index will be skipped)
5) Is the value added to the vertex index before indexing into the vertex buffer. I believe this means that it's offset into the vertex buffer before we start using it. Mean if we have 1 we skip the first vertex.
6) The offset for instancing
# References
##### Main Notes
[[Creating the index buffer with staging buffers]]
#### Source Notes
[[Vertex buffer]]
