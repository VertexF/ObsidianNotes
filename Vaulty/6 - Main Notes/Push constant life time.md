2026-02-07 10:08
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# Push constant life time

You are totally allow to bind both 2 different pipeline and then swap between 
```cpp
// different ranges and therefore not compatible layouts
VkPipelineLayout layout_graphics; // VK_SHADER_STAGE_FRAGMENT_BIT
VkPipelineLayout layout_compute;  // VK_SHADER_STAGE_COMPUTE_BIT

// vkBeginCommandBuffer()
vkCmdBindPipeline(pipeline_graphics); // layout_graphics
vkCmdBindPipeline(pipeline_compute);  // layout_compute

vkCmdPushConstants(layout_graphics); // VK_SHADER_STAGE_FRAGMENT_BIT
// Still valid as the last pipeline and push constant for graphics are compatible
vkCmdDraw();

vkCmdPushConstants(layout_compute); // VK_SHADER_STAGE_COMPUTE_BIT
vkCmdDispatch(); // valid
// vkEndCommandBuffer()
```

You are totally allowed to bind pipelines that don't use a push constants at any point in any order.
```cpp
// vkBeginCommandBuffer()
vkCmdPushConstants(layout_0);
vkCmdBindPipeline(pipeline_b); // non-compatible with layout_0
vkCmdBindPipeline(pipeline_a); // compatible with layout_0
vkCmdDraw(); // valid
// vkEndCommandBuffer()

// vkBeginCommandBuffer()
vkCmdBindPipeline(pipeline_b); // non-compatible with layout_0
vkCmdPushConstants(layout_0);
vkCmdBindPipeline(pipeline_a); // compatible with layout_0
vkCmdDraw(); // valid
// vkEndCommandBuffer()

// vkBeginCommandBuffer()
vkCmdPushConstants(layout_0);
vkCmdBindPipeline(pipeline_a); // compatible with layout_0
vkCmdBindPipeline(pipeline_b); // non-compatible with layout_0
vkCmdDraw(); // INVALID
// vkEndCommandBuffer()
```

This is how you want to update your push constants if something is happening between draw calls. It all works as you would expect.
```c++
// vkBeginCommandBuffer()
vkCmdBindPipeline();
vkCmdPushConstants(offset: 0, size: 16, value = [0, 0, 0, 0]);
vkCmdDraw(); // values = [0, 0, 0, 0]

vkCmdPushConstants(offset: 4, size: 8, value = [1 ,1]);
vkCmdDraw(); // values = [0, 1, 1, 0]

vkCmdPushConstants(offset: 8, size: 8, value = [2, 2]);
vkCmdDraw(); // values = [0, 1, 2, 2]
// vkEndCommandBuffer()
```
# References
##### Main Notes
[[Using a push constants]]
[[Push constant with offset data]]
#### Source Notes
