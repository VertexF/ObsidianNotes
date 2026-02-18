# Reference https://docs.vulkan.org/samples/latest/samples/performance/multi_draw_indirect/README.html

The idea with GPU driven rendering and indirect draw calls is to offload the draw call generation to the GPU. This allows you to cull geometry on the GPU, which can then reduce the amount of draw calls because we are allowing the GPU to generate draw calls.
##### Draw Call Generation
A common idea when it comes to rendering, you iterate through each model to draw in a scene and bind there resource such as vertex buffers, index buffer and descriptors before the draw call. One thing to note is that this isn't free in command buffer generation such as **vkCmdBindVertexBuffer** and rendering.

You could use indirect draw calls instead introduces in Vulkan 1.2 which queries and GPU buffer to get it's draw paramters.

You have two advantages here:
1) Draw calls can be generating on the GPU such as in computer (which can do additional work to cull things.)
2) Using an array of draw calls can be called once, reducing the command buffer overhead.

When you want to use an indirect draw call, you first need to use the struct **VkDrawIndexedIndirectCommand**, this provide the information of the vertices and indices to draw. Since you an offset into an index buffer and vertex buffer using the **VkDrawIndexedIndirectCommand** struct, you can make a HUGE index buffer that contains a bunch of models in a scene bind that once, even if the total number of vertices exceed 2^16 or 2^32 in that draw call. This can **ONLY WORK IF** each portion of the index buffer is zero-indexing. Meaning that if each model has say an index buffer of (0, 5, 6, ....) and (99, 0, 120, ...) they can live in one index buffer IF the index buffer counts starting from 0, not 1.

Okay this is important, you don't create the **VkDrawIndexedIndirectCommand** on the CPU but on the GPU that's the point. You pass up the **VkDrawIndexedIndirectCommand** in an indirect draw call which is filled in a shader.

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
3) The offset of that draw command buffer which in this case is the **indirect** part of the **MeshDrawCommand** which is 5 uint32_t in size.
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
