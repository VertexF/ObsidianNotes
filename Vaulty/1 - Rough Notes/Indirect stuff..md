When you want to use an indirect count buffer so you can count how many draw calls you have. This allows you to create a buffer that is filled up on the GPU full of draw command. 

Both buffers will get there data from GPU, the count buffer will get it's data from a `vkCmdFillBuffer` Meaning that we need to make sure that it's`VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT` is set up in the `memoryInfo.requiredFlags` of `VmaAllocationCreateInfo`. You don't want anything host related in the `VmaAllocationCreateInfo` as that doesn't make sense.

The count buffer should only contain a uint32_t number so 4 bytes of data as it's only increasing a number to keep track of the index when we need to create a new draw call, for our draw buffer.

After we have filled the count buffer we need to transition the buffer to be readable by a shader. 

```c++
BufferHandle indirectCountBufferHandle = INVALID_BUFFER;
{
	BufferHandle handle = { buffers.obtainResource() };
	if (handle.index == INVALID_INDEX)
	{
		exit(-1);
	}
	
	Buffer* buffer = accessBuffer(handle);
	buffer->name = "Indirect Draw Count Buffer";
	buffer->size = 4;
	buffer->typeFlags = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT | VK_BUFFER_USAGE_INDIRECT_BUFFER_BIT;
	buffer->handle = handle;
	buffer->globalOffset = 0;
	
	VkBufferCreateInfo bufferInfo{};
	bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
	bufferInfo.usage = VK_BUFFER_USAGE_TRANSFER_DST_BIT | VK_BUFFER_USAGE_STORAGE_BUFFER_BIT | VK_BUFFER_USAGE_INDIRECT_BUFFER_BIT;
	bufferInfo.size = 4;
	
	VmaAllocationCreateInfo memoryInfo{};
	memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;
	memoryInfo.requiredFlags = VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT;
	
	VmaAllocationInfo allocationInfo{};
	check(vmaCreateBuffer(VMAAllocator, &bufferInfo, &memoryInfo, &buffer->vkBuffer, &buffer->vmaAllocation, &allocationInfo));
	
	setResourceName(VK_OBJECT_TYPE_BUFFER, reinterpret_cast<uint64_t>(buffer->vkBuffer), "Indirect Draw Count Buffer");
	buffer->vkDeviceMemory = allocationInfo.deviceMemory;
	
	indirectCountBufferHandle = handle;
}

//Other stuff happens here.

Buffer* buffer = accessBuffer(indirectCountBufferHandle);

VkBufferMemoryBarrier2 bufferBarrier{};
bufferBarrier.sType = VK_STRUCTURE_TYPE_BUFFER_MEMORY_BARRIER_2;
bufferBarrier.srcStageMask = VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT;
bufferBarrier.srcAccessMask = VK_ACCESS_INDIRECT_COMMAND_READ_BIT;
bufferBarrier.dstStageMask = VK_PIPELINE_STAGE_TRANSFER_BIT;
bufferBarrier.dstAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;
bufferBarrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
bufferBarrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
bufferBarrier.buffer = &buffer->vkBuffer;
bufferBarrier.offset = 0;
bufferBarrier.size = VK_WHOLE_SIZE;

VkDependencyInfo dependencyInfo{};
dependencyInfo.sType = VK_STRUCTURE_TYPE_DEPENDENCY_INFO;
dependencyInfo.bufferMemoryBarrierCount = 1;
dependencyInfo.pBufferMemoryBarriers = &bufferBarrier;
dependencyInfo.imageMemoryBarrierCount = 0;
dependencyInfo.pImageMemoryBarriers = nullptr;

vkCmdPipelineBarrier2(commandBuffer, &dependencyInfo);

vkCmdFillBuffer(commandBuffer, buffer->vkBuffer, 0, 4, 0);
```

We also need to create the draw command buffer which should empty as this is going to be filled within a compute shader, that's why we need a bunch work eariler to fill a buffer with 0's because we are going use that in a compute shader to go through each element within the draw command and exclude things we don't need, this is the power of fully GPU driven rendering.

```c++
BufferHandle indirectBufferHandle = INVALID_BUFFER;
{
	BufferHandle handle = { buffers.obtainResource() };
	if (handle.index == INVALID_INDEX)
	{
		exit(-1);
	}
	
	Buffer* buffer = accessBuffer(handle);
	buffer->name = "Indirect Draw Buffer";
	//I don't know why yet you would want to make a draw command this large.
	buffer->size = 128 * 1024 * 1024;
	buffer->typeFlags = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT | VK_BUFFER_USAGE_INDIRECT_BUFFER_BIT;
	buffer->handle = handle;
	buffer->globalOffset = 0;
	
	VkBufferCreateInfo bufferInfo{};
	bufferInfo.sType = VK_STRUCTURE_TYPE_BUFFER_CREATE_INFO;
	bufferInfo.usage = VK_BUFFER_USAGE_STORAGE_BUFFER_BIT | VK_BUFFER_USAGE_INDIRECT_BUFFER_BIT;
	//I don't know why yet you would want to make a draw command this large.
	bufferInfo.size = 128 * 1024 * 1024;
	
	VmaAllocationCreateInfo memoryInfo{};
	memoryInfo.usage = VMA_MEMORY_USAGE_AUTO;
	memoryInfo.requiredFlags = VK_MEMORY_PROPERTY_DEVICE_LOCAL_BIT;
	
	VmaAllocationInfo allocationInfo{};
	check(vmaCreateBuffer(VMAAllocator, &bufferInfo, &memoryInfo, &buffer->vkBuffer, &buffer->vmaAllocation, &allocationInfo));
	
	setResourceName(VK_OBJECT_TYPE_BUFFER, reinterpret_cast<uint64_t>(buffer->vkBuffer), "Indirect Draw Count Buffer");
	buffer->vkDeviceMemory = allocationInfo.deviceMemory;
	
	indirectBufferHandle = handle;
}
```

After this we need to transfer both buffers to be ready for into from transfer mode for count buffer and indirect mode for the indirect draw buffer, into compute shader mode so they can be read and written in the compute shader.

```c++
    VkBufferMemoryBarrier2 bufferComputeBarrier{};
    bufferComputeBarrier.sType = VK_STRUCTURE_TYPE_BUFFER_MEMORY_BARRIER_2;
    bufferComputeBarrier.srcStageMask = VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT | VK_PIPELINE_STAGE_VERTEX_SHADER_BIT;
    bufferComputeBarrier.srcAccessMask = VK_ACCESS_INDIRECT_COMMAND_READ_BIT | VK_ACCESS_SHADER_READ_BIT;
    bufferComputeBarrier.dstStageMask = VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT;
    bufferComputeBarrier.dstAccessMask = VK_ACCESS_SHADER_WRITE_BIT;
    bufferComputeBarrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    bufferComputeBarrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    bufferComputeBarrier.buffer = indirectBuffer->vkBuffer;
    bufferComputeBarrier.offset = 0;
    bufferComputeBarrier.size = VK_WHOLE_SIZE;

    VkBufferMemoryBarrier2 bufferComputeCountBarrier{};
    bufferComputeCountBarrier.sType = VK_STRUCTURE_TYPE_BUFFER_MEMORY_BARRIER_2;
    bufferComputeCountBarrier.srcStageMask = VK_PIPELINE_STAGE_TRANSFER_BIT;
    bufferComputeCountBarrier.srcAccessMask = VK_ACCESS_TRANSFER_WRITE_BIT;
    bufferComputeCountBarrier.dstStageMask = VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT;
    bufferComputeCountBarrier.dstAccessMask = VK_ACCESS_SHADER_READ_BIT | VK_ACCESS_SHADER_WRITE_BIT;
    bufferComputeCountBarrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    bufferComputeCountBarrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
    bufferComputeCountBarrier.buffer = indirectBufferCount->vkBuffer;
    bufferComputeCountBarrier.offset = 0;
    bufferComputeCountBarrier.size = VK_WHOLE_SIZE;

    VkBufferMemoryBarrier2 computeBarriers[] =
    {
        bufferComputeBarrier,
        bufferComputeCountBarrier
    };

    VkDependencyInfo dependencyComputeInfo{};
    dependencyComputeInfo.sType = VK_STRUCTURE_TYPE_DEPENDENCY_INFO;
    dependencyComputeInfo.bufferMemoryBarrierCount = 2;
    dependencyComputeInfo.pBufferMemoryBarriers = computeBarriers;
```

What we are doing above is the required steps to make the compute shader be able to read the count buffer which will be used to generate draw calls. 

Now with all our memory in the correct format we need to run the `vkCmdDispatch` to actually do the work of creating the draw commands.

You can set the number of draws you want to create. That's what the `drawCount` is for. We also are using or push constant to access these buffers via a BDA.

It's also important to avoid having to do SPIR-V reflection to look up the number of threads, we are just keeping the relationship between the `vkCmdDispatch` and the number threads ourselves.

```c++
uint32_t drawCount = 1000000;
uint32_t computeLocalXID = 32;
{ //Fake Render loop.
	vkCmdBindPipeline(commandBuffer, VK_PIPELINE_BIND_POINT_COMPUTE, pipeline);
	
	Buffer* indirectBuffer = accessBuffer(indirectBufferHandle);
	Buffer* indirectBufferCount = accessBuffer(indirectCountBufferHandle);
	
	pushConstants.indirectAddress = indirectBuffer->bufferAddress;
	pushConstants.indirectCountAddress = indirectBufferCount->bufferAddress;
	
	vkCmdPushConstants(gpuCommands->vkCommandBuffer, vkComputePipeline, VK_SHADER_STAGE_VERTEX_BIT, 0, sizeof(pushConstants), &pushConstants);
	
	uint32_t numberOfXThreads = (drawCount + computeLocalXID - 1) / computeLocalXID;
	
	vkCmdDispatch(commandBuffer, numberOfXThreads, 1, 1);
}
```

The next step is creating the compute shader that fills up the `VkDrawIndexedIndirectCommand indirect;` or in our case `VkDrawIndirectCommand indirect;`

```c
struct DrawCommand
{
	//If you not using a mesh/task approach to this you don't need to store the drawID.
    //uint drawID;

    uint vertexCount;
    uint instanceCount;
    uint firstVertex;
    uint firstInstance;
};

layout(scalar, buffer_reference, buffer_reference_align = 8) writeonly buffer DrawCommandData
{
    DrawCommand drawCommands[];
};

layout(scalar, buffer_reference, buffer_reference_align = 8) buffer DrawCommandCount
{
    uint drawCommandCount;
};

layout(scalar, push_constant) uniform entityIndex
{
    QuadPositionData quadPositionsReference;
    SceneBuffer2DData scene2D;
};

layout(local_size_x_id = 32, local_size_y_id = 1, local_size_z_id = 1) in;

void main()
{
	//TODO decided and write the particle logic
	//Do the particles logic here + view frustum cull here.

    uint dci = atomicAdd(drawCommandCount, 1);

    drawCommands[dci].vertexCount = 6;
    drawCommands[dci].instanceCount = 1;
    drawCommands[dci].firstVertex = 0;
    drawCommands[dci].firstInstance = 0;
}
```

This example here is going of a compute shader that just fills up the command and does nothing at all. In the comment in void main I am assuming you are eventually going to do stuff. Next step is transitioning the buffer we have written to to be readable by draw call.

```c++
Buffer* indirectBuffer = accessBuffer(indirectBufferHandle);
Buffer* indirectCountBuffer = accessBuffer(indirectCountBufferHandle);

VkBufferMemoryBarrier2 bufferIndirectBarrier{};
bufferBarrier.sType = VK_STRUCTURE_TYPE_BUFFER_MEMORY_BARRIER_2;
bufferBarrier.srcStageMask = VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT;
bufferBarrier.srcAccessMask =  VK_ACCESS_SHADER_WRITE_BIT;
bufferBarrier.dstStageMask = VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT | VK_PIPELINE_STAGE_VERTEX_SHADER_BIT;
bufferBarrier.dstAccessMask = VK_ACCESS_INDIRECT_COMMAND_READ_BIT | VK_ACCESS_SHADER_READ_BIT;
bufferBarrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
bufferBarrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
bufferBarrier.buffer = &indirectBuffer->vkBuffer;
bufferBarrier.offset = 0;
bufferBarrier.size = VK_WHOLE_SIZE;

VkBufferMemoryBarrier2 bufferIndirectCountBarrier{};
bufferBarrier.sType = VK_STRUCTURE_TYPE_BUFFER_MEMORY_BARRIER_2;
bufferBarrier.srcStageMask = VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT;
bufferBarrier.srcAccessMask =  VK_ACCESS_SHADER_WRITE_BIT;
bufferBarrier.dstStageMask = VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT;
bufferBarrier.dstAccessMask = VK_ACCESS_INDIRECT_COMMAND_READ_BIT;
bufferBarrier.srcQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
bufferBarrier.dstQueueFamilyIndex = VK_QUEUE_FAMILY_IGNORED;
bufferBarrier.buffer = &indirectCountBuffer->vkBuffer;
bufferBarrier.offset = 0;
bufferBarrier.size = VK_WHOLE_SIZE;

VkBufferMemoryBarrier2 barriers[] =
{
	bufferIndirectBarrier,
	bufferIndirectCountBarrier
};

VkDependencyInfo dependencyInfo{};
dependencyInfo.sType = VK_STRUCTURE_TYPE_DEPENDENCY_INFO;
dependencyInfo.bufferMemoryBarrierCount = 2;
dependencyInfo.pBufferMemoryBarriers = barriers;
```

First thing we must mention is that we are going to increment by 1 going over indirect buffer count, in a compute shader when we have a condition for a draw call. We need to limit the amount of draw calls we have for a given thing.

```c++
Buffer* indirectBuffer = accessBuffer(indirectBufferHandle);
Buffer* indirectCountBuffer = accessBuffer(indirectCountBufferHandle);

commandBuffer.bindPipeline(pipeline2D);

scene2d.project = camera3D.projection;
scene2d.view = camera3D.view;

commandBuffer.bindlessDescriptorSet(0);

Buffer* quadPositionBuffer = gpu->accessBuffer(positionalBDAHandle[gpu->currentFrame]);
Buffer* sceneBuffer = gpu->accessBuffer(sceneBDAHandle);

vmaCopyMemoryToAllocation(gpu->VMAAllocator, quadData.data, quadPositionBuffer->vmaAllocation, 0, sizeof(QuadPositionData));
vmaCopyMemoryToAllocation(gpu->VMAAllocator, &scene2d, sceneBuffer->vmaAllocation, 0, sizeof(SceneData2D));

PushConstant pushConstants{};
pushConstants.quadPostionAddress = quadPositionBuffer->bufferAddress;
pushConstants.sceneAddress = sceneBuffer->bufferAddress;

vkCmdPushConstants(commandBuffer.vkCommandBuffer, commandBuffer.currentPipeline->vkPipelineLayout, VK_SHADER_STAGE_VERTEX_BIT, 0, sizeof(pushConstants), &pushConstants);

vkCmdDrawIndirectCount(commandBuffer, indirectBuffer->vkBuffer, sizeof(MeshDrawCommand), indirectCountBuffer->vkBuffer, 0, 1000000, sizeof(MeshDrawCommand));
```

Now when you run the indirect `vkCmdDrawIndirectCount`you will get all the regular stuff that you would get with a regular draw call include `gl_InstanceIndex` it's just indirect nothing else changes. So if you have 1000000 draws calls and 1000000 instances you have 1000000 x 1000000 objects in the scene. If you need to access the draws in the shader you do `gl_DrawIDARB`

Just make sure you bind the graphics pipeline that correct corrisponds to the compute set up for indirect draw calls.