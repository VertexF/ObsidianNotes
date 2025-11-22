2025-11-21 19:18
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# Binding a descriptor set to use at draw time

After creating the descriptor sets you'll need to run this before any **vkCmdDraw...** 

```c++
vkCmdBindDescriptorSets(commandBuffers[currentFrame], VK_PIPELINE_BIND_POINT_GRAPHICS, pipelineLayout, 0, 1, &descriptorSets[currentFrame], 0, nullptr);
```

2) Descriptor sets can be used in compute pipelines as well as graphics pipelines so set that with **VK_PIPELINE_BIND_POINT_GRAPHICS** 
3) Next you need to specify the pipeline layout that the descriptor set is based on.
4) The next three are to do with the index of the descriptor. So the forth argument is the telling vulkan what the first index is.
5) This tells vulkan how many to bind at a time.
6) Next is the actually array of descriptor sets to bind. 
7) The next two are to do with dynamic descriptors and the offset into them. We aren't using them so the dynamic count is 0
8) The dynamic descriptor array of uint32_t which are the offsets themselves should be nullptr if you're not using this feature.
# References
##### Main Notes
[[Drawing with an index buffer]]
[[Creating descriptor sets]]
#### Source Notes
[[Uniform buffers]]