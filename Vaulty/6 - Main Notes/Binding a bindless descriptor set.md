2026-02-27 09:53
Status: #baby 
Tags: [[vulkan]] [[vulkan descriptors]] [[vulkan textures]]
# Binding a bindless descriptor set

Yes, even though it's called bindless you still bind a bindless descriptor set. This is actually exactly the same as when you're [[Binding a descriptor set to use at draw time]] 

The difference here is when you bind a texture descriptor set. You do this once per frame as you're binding an array of textures all at once, rather than at every draw call like with non-bindless textures

```c++
bool running = true;
while(running)
{
	//Event code here!
	
	vkCmdBindDescriptorSets(vkCommandBuffer, VK_PIPELINE_BIND_POINT_GRAPHICS, currentPipeline->vkPipelineLayout, 1, 1, &device->bindlessDescriptorSet, 0, nullptr);
	
	//Draw call here!
}
```

It's important note here is that the set index in 4th argument `1` needs to match your bindless descriptor set [[Setting up bindless textures in a shader|in your shader.]]
# References
##### Main Notes
[[Binding a descriptor set to use at draw time]] 
[[Setting up bindless textures in a shader]]
[[Updating a bindless descriptor set]]
#### Source Notes
[[Bindless - Mastering Graphics Programming with Vulkan]]
