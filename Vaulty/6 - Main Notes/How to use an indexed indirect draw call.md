2026-02-13 10:31
Status: #baby 
Tags: [[vulkan]] [[draw calls]]
# How to use an indexed indirect draw call

We are assuming you have the foundational understanding here of [[Setting up an indexed indirect draw call]]

These are the steps how the indirect buffer would work for a compute shader that builds up the **VkDrawIndexedIndirectCommand**. It first writes out the buffer in the computer and is read in on the vertex shader.

It's important to note that I've left in the barriers here because it's important because we are writing to a buffer. 
```c++
VkBufferMemoryBarrier2 fillBarriers[] =
{
    bufferBarrier(dcb.buffer,
    VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT | rasterizationStage, VK_ACCESS_INDIRECT_COMMAND_READ_BIT | VK_ACCESS_SHADER_READ_BIT,
VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT, VK_ACCESS_SHADER_WRITE_BIT)
};

//Set up stuff to do with getting ready for the compute shader.

vkCmdDispatch(commandBuffer, getGroupCount(uint32_t(draws.size()), drawcullCS.localSizeX), 1, 1);

VkBufferMemoryBarrier2 cullBarriers[] =
{
    bufferBarrier(dcb.buffer,
VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT, VK_ACCESS_SHADER_WRITE_BIT,
VK_PIPELINE_STAGE_DRAW_INDIRECT_BIT | rasterizationStage, VK_ACCESS_INDIRECT_COMMAND_READ_BIT | VK_ACCESS_SHADER_READ_BIT)
};
```

This is a very vague compute shader but it's important to note to look at the bottom of the shader, we are writing to a buffer that's **writeonly** 
```c
struct MeshDrawCommand
{
    uint drawID;

    uint indexCount;
    uint instanceCount;
    uint firstIndex;
    uint vertexOffset;
    uint firstInstance;
};

struct MeshDraw
{
	//Contains material data for PBR here.

    //== meshes[meshIndex].vertexOffset, help data locality in the mesh shader.
    uint vertexOffset;
};

layout(set = 0, std430, binding = 0) writeonly buffer DrawCommands
{
    MeshDrawCommand drawCommands[];
};

layout(set = 0, std430, binding = 1) readonly buffer MeshDraws
{
    MeshDraw meshDraws[];
};

void main()
{
	uint meshInstanceIndex = gl_GlobalInvocationID.x;
	MeshDraw meshDraw = meshDraws[meshDrawIndex];

	//Stuff happends here to cull geometry or something
    drawCommands[drawIndex].drawID = meshInstanceIndex;
    drawCommands[drawIndex].indexCount = 0;
    drawCommands[drawIndex].instanceCount = 1;
    drawCommands[drawIndex].firstIndex = 0;
    drawCommands[drawIndex].vertexOffset = meshDraw.vertexOffset;
    drawCommands[drawIndex].firstInstance = 0;
}
```

Next based on the on the command buffer we have written out we need to specify the offset of that command buffer we are going to upload to a shader. 
```c++
struct MeshDrawCommand
{
    uint32_t drawID;
    VkDrawIndexedIndirectCommand indirect; // 5 uint32_t
};

vkCmdDrawIndexedIndirect(commandBuffer, dcb.buffer, offsetof(MeshDrawCommand, indirect), uint32_t(draws.size()), sizeof(VkDrawIndexedIndirectCommand));
```

The arguements go:
1) command buffer,
2) The filled draw command buffer.
3) The offset of that draw command buffer which in this case is the **indirect** part of the **MeshDrawCommand** struct which is 5 uint32_t in size.
4) The number of draw calls.
5) The stride which is needed to stride over those 5 uint32_t, so we set the stride with the sizeof that draw command buffer.

Finally we can use the command buffer here with **readonly** buffer.
```c
layout(set = 0, binding = 3) readonly buffer VisibleMeshInstances
{
    MeshDrawCommand drawCommands[];
};

void main()
{
	uint meshInstanceIndex = drawCommands[gl_DrawIDARB].drawID;
	
	//Fetch the data with the modelInstanceIndex.
	//Do the vertex shader work here.
}
```
# References
##### Main Notes
[[Setting up an indexed indirect draw call]]
[[What are indirect draw calls]]
[[Advantages of indirect draw calls]]
#### Source Notes
[[GPU Rendering and Multi-Draw Indirect]]