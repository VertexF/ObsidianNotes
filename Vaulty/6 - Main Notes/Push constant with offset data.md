2026-02-07 09:58
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]]
# Push constant with offset data

If you want to offset into data within your push constant, you'll need set up the offset at pipeline layout time.

```c++
VkPushConstantRange range{};
range.stageFlags = VK_SHADER_STAGE_VERTEX_BIT;
range.offset = 32;
range.size = 4;

VkPipelineLayoutCreateInfo pipelineLayoutInfo{};
pipelineLayoutInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_LAYOUT_CREATE_INFO;
pipelineLayoutInfo.pSetLayouts = vkLayout;
pipelineLayoutInfo.setLayoutCount = creation.numActiveLayouts;
pipelineLayoutInfo.pushConstantRangeCount = 1;
pipelineLayoutInfo.pPushConstantRanges = &range;
```

You'll have to layout like this within a shader like this.

```c
layout(push_constant, std140) uniform entityIndex
{
    layout(offset = 32) uint index;
};
```

This image example the layout process really clearly

![[push_constant_offset.png]]

```c++
for (uint32_t i = 0; i < totalDucks; ++i)
{
    uint32_t offset = 32;
    uint32_t size = 4;
    vkCmdPushConstants(vkCommandBuffer, vkPipelineLayout, VK_SHADER_STAGE_VERTEX_BIT, offset, size, &i);
    
    //...other stuff
}
```
# References
##### Main Notes
[[Using a push constants]]
[[Push constant life time]]
#### Source Notes
