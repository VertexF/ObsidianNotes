2026-04-29 12:04
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# Split memory barriers

This concept is very similar to [[Building a execution dependency chain]] as we can also do the memory transfer over two different memory barriers. By making memory **available** in one memory barrier and make memory **visible** in another memory barrier.

If you're using dynamic rendering or render passes without subpass dependencies. When ordering your work in queues for a frame this is what you'll need to do for your colour attachments. With work in the middle running the shaders.

Here is an example of what a split memory barrier looks like, for a situation were a computer shader is writing to an SSBO and we later read from it in another compute shader. 
```c++
vkCmdDispatch();//writes to an SSBO, VK_ACCESS_SHADER_WRITE_BIT

vkCmdPipelineBarrier(srcStageMask = COMPUTE, dstStageMask = TRANSFER, srcAccessMask = SHADER_WRITE_BIT, dstAccessMask = 0)

vkCmdPipelineBarrier(srcStageMask = TRANSFER, dstStageMask = COMPUTE, srcAccessMask = 0, dstAccessMask = SHADER_READ_BIT)

vkCmdDispatch();//read from the same SSBO, VK_ACCESS_SHADER_READ_BIT
```

Note that the stage masks cannot be 0 and the access masks can be 0.
# References
##### Main Notes
[[Building a execution dependency chain]]
[[How GPU memory works]]
[[What are the access masks]]
[[The difference between source and destination stage mask]]
[[Making non-global pipeline barriers]]
[[In-queue execution pipeline barriers]]
[[The pipeline stages]]
[[Vulkan command categories]]
[[Implicit synchronisation guarantees]]
[[Basic queue synchronisation]]
[[Vulkan command overlap]]
#### Source Notes
[[Vulkan synchronisation explained]]
