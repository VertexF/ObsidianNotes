2025-10-25 15:49
Status: #baby 
Tags: [[vulkan]] [[vulkan graphic pipeline]]
# Setting up the dynamic states

When you're creating a graphics pipeline you don't want everything to do static. There are three things that don't have to static.
1) Viewport size and scissor size
2) Line width
3) Blend constants

These are things that need to be created with vulkan commands at draw time. If you want a resizable window you'll need to block out the make viewport setup and scissor setup when creating the fixed function part of the graphics pipeline. 

You do this with the **VkPipelineDynamicStateCreateInfo** struct. This is the example for resizeable viewports and scissor.

```c++
std::vector<VkDynamicState> dynamicStates = 
{
	VK_DYNAMIC_STATE_VIEWPORT,
	VK_DYNAMIC_STATE_SCISSOR
};

VkPipelineDynamicStateCreateInfo dynamicState{};
dynamicState.sType = VK_STRUCTURE_TYPE_DEVICE_CREATE_INFO;
dynamicState.dynamicStateCount = uint32_t(dynamicStates.size());
dynamicState.pDynamicStates = dynamicStates.data();
```
# References
##### Main Notes
[[What is the graphics pipeline]]
#### Source Notes
[[Vulkan-Tutorial]]