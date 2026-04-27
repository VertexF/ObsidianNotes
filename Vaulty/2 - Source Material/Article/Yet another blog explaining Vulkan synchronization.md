# Reference https://themaister.net/blog/2019/08/14/yet-another-blog-explaining-vulkan-synchronization/

Synchronisation is hard. If you want to debug synchronisation in vulkan you can turn it on in your Vulkan Configuration tool. This will allow validation layers to catch mistakes you're making.
##### Pipeline barriers
The whole point of these is to top overlapping of resources and/or memory from action commands. The memory exists at pipeline stage, meaning we set up a barrier between action commands on these stages.
##### Implicit Synchronisation Guarantees
When talking about synchronisation between commands it's important to note that vulkan has implicit synchronisation guarantees. This means that command recorded to a command buffer AND submission to a `vkQueueSubmit` ordering matters.
###### The Vulkan queue
Synchronsation of queues in vulkan is basically the same as 1 queue. So a queue is just an abstraction of vulkan blasting through commands. 
##### Command buffer misconceptions
All commands are global to a queue so synchronisation to anything within that queue is global to all commands.
##### Commands overlap
When you're recording command to a command buffer, the order is important and preserved. The execution of commands in a queue is just a stream of commands from start to finish. Command will fire off one after another in sequence **BUT** end out of order.

This means that the actual execution of the commands is out of order, because starting a command isn't the same an executed command.

The graphics driver may overlap command like vkDraw and vkDispatch action commands. This is were you would might need a barrier if the vkDispatch is dependent on vkDraw, but remember all action commands will finish executing at a unspecified time meaning that you'll like need more memory barriers.

Framebuffer operations inside a render pass happen in order which is an expection.
###### Pipeline Stages
Action command which are everything other than command buffer state change, synchronisation and indirect commands consist of multiple operations, which go pipeline stages. The pipeline stages that get executed to depend on the action command and the state of the command buffer.

In the pipeline staged themselves aren't complete and strictly followed by the actual work. This is because some stages can be merged and in the `VK_PIPELINE_STAGE` some stages are left out.

You also have three special stages that have special access OR combinding stages.
1) `VK_PIPELINE_STAGE_HOST_BIT`
2) `VK_PIPELINE_STAGE_ALL_COMMANDS_BIT`
3) `VK_PIPELINE_STAGE_ALL_GRAPHICS_BIT`

A command you submit goes from `VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT` to `VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT` every single one. So we are adding synchronisation to the pipeline line stages for all commands.
##### The mysterious TOP_OF_PIPE_BIT and BOTTOM_OF_PIPE_BIT stages. 
`VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT` is actually were the command gets parsed and `VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT` is were the comamnd returns. In **synchronisation2** `TOP_OF_PIPE_BIT` is `NONE` and `BOTTOM_OF_PIPE_BIT` and `ALL_COMMANDS`.
##### In-queue execution barriers
A execution barrier is a subset of memory barriers. An execution barrier is a pipeline barrier.

```c++
void vkCmdPipelineBarrier(
VkCommandBuffer commandBuffer,
VkPipelineStageFlags srcStageMask,
VkPipelineStageFlags dstStageMask,
VkDependencyFlags dependencyFlags,
uint32_t memoryBarrierCount,
const VkMemoryBarrier* pMemoryBarriers,
uint32_t bufferMemoryBarrierCount,
const VkBufferMemoryBarrier* pBufferMemoryBarriers,
uint32_t imageMemoryBarrierCount,
const VkImageMemoryBarrier*pImageMemoryBarriers);
```

If we ignore the all the memory barrier flags in the function above you just have the **srcStageMask** and the **dstStageMask**. 

The **srcStageMask** and the **dstStageMask** are at the heart of synchronisation, in the command stream we have a before **srcStageMask** and **dstStageMask** for after.
##### The difference between the srcStageMask and dstStageMask
VkPipelineStageFlags **srcStageMask** and **dstStageMask** can be thought of one being the "consumer" stage (**srcStageMask**) and "producer" stage (**dstStageMask**). By specifying the source and target stages you tell the driver what operations need to finish before the transition can execute (the **srcStageMask**) and what can't have started yet (the **dstStageMask**).

For example if you have this memory barrier set up like this you have.

```c++
vkCmdPipelineBarrier(
    commandBuffer,
    VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT, // source stage
    VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT,    // destination stage
    /* remaining parameters omitted */);
```

When you add this memory barrier you are saying that every frame in flight needs to finish and no work must start before the transition happens. Effectively the barrier will wait for everything to finish and block work from starting. This isn't very good because you have a pipeline bubble.

Let use a better example lets say you want a vertex shader that stores data via and`imageStore` and you want that to be consumed by a compute shader. In this case wouldn't want to wait for the fragment shader to finish THEN do the computer shader as the fragment shader might take a long time. We would want the vertex shader to finish then immediately followed by the compute shader.

The way to express this is have the **srcStageMask** to be `VK_PIPELINE_STAGE_VERTEX_SHADER_BIT` and the consumer `VK_PIPELINE_COMPUTE_SHADER_BIT` be the **dstStageMask**

```c++
vkCmdPipelineBarrier(
commandBuffer,
VK_PIPELINE_VERTEX_SHADER_BIT, // source stage
VK_PIPELINE_COMPUTE_SHADER_BIT, // destination stage
/* remaining parameters omitted */);
```

The idea here is that we have 2 action commands which execute at different pipeline stages. They execute the green parts or may just skip them. 
![[vulkan-good-barrier-1024x771.Bl25au61.jpg]]

So in the first command we are skipping over the compute shader so the **dstStageMask** doesn't affect the current action command. This is because we don't need to wait on anything as nothing is going to happen. For the first command the compute shader is a noop.
##### IMPORTANT
This is extremely important that you don't get recording commands or submitting a queue mixed up with work being done which reference to an action running through it's pipeline stages to execute. 
##### The srcStageMask
```c++
//We ARE dealing with this!
vkCmdDispatch(VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT)
//We are NOT dealing with this!
vkCmdCopyBuffer(VK_PIPELINE_STAGE_TRANSFER_BIT)
//We ARE dealing with this!
vkCmdDispatch(VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT)

//Look how the VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT is the same as the vkCmdDispatch
vkCmdPipelineBarrier (srcStageMask = VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT)
```

So in the above example we have 3 action commands. The **vkCmdDispatch** is going to execute on `VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT` which tells the driver what operations need to finish before the transition can execute. The **vkCmdCopyBuffer** has nothing in the `VK_PIPELINE_STAGE_COMPUTE_SHADER_BIT` so there is nothing to wait on.

Even if we took these command and submitted into 4 different `vkQueueSubmit()` we would still consider them under the same commands for synchronisation.

The **srcStageMask** restricts the scope of what we are waiting. You can wait on more than one stage at a time by **OR-ing** stages together.
##### The dstStageMask
This is the second half of the barrier. 

If we set the **dstStageMask** is set to something like `VK_PIPELINE_STAGE_VERTEX_SHADER_BIT` if a fragment shader is going to run on the GPU on the same action command like a vkCmdDraw after the vertex shader stage it can. We will only pause a command once it hits `VK_PIPELINE_STAGE_VERTEX_SHADER_BIT`. 
##### Super basic example
Here is an example of adding a memory barrier. Note that we are **NOT** looping through **vkCmdDispatch**

```c++
1 vkCmdDispatch
2 vkCmdDispatch
3 vkCmdDispatch
4 vkCmdPipelineBarrier(srcStageMask = COMPUTE, dstStageMask = COMPUTE)
5 vkCmdDispatch
6 vkCmdDispatch
7 vkCmdDispatch
```

With this barrier, the “before” set is commands {1, 2, 3}. The “after” set is {5, 6, 7}. A possible execution order here could be:
- #3
- #2
- #1
- #7
- #6
- #5

If you want to dig into the examples to make sure to look through the example https://github.com/KhronosGroup/Vulkan-Docs/wiki/Synchronization-Examples to make sure you are doing what you want.
##### Events aka Split barriers
With events you can have a barrier that overlap work. You can use **VkEvents** to split up a memory barrier. The idea here is that the command inbetween that can overlap with other command before or after the set of commands.

For example if I have a `vkCmd1` and `vkCmd2` which have dependency on each other. However while `vkCmd2` waits for `vkCmd1` to finish, we could have `vkCmdA` and `vkCmdB` while while the pause is happening.

```c++
vkCmd1;
vkCmdSetEvent(X, VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT);
vkCmdA;
vkCmdB;
vkCmdSetEvent(X, VK_PIPELINE_STAGE_COLOR_FRAGMENT_SHADER_BIT);
vkCmd2;
```

Now `vkCmdA` and `vkCmdB` run whenever they like. 

Not all GPUs take advantage of this feature but it can be very important for advanced compute work. The reason being if there is a gap that the barrier you can through something else that fills up the GPU with work.
##### Execution dependency chain.
With these barriers you can build up a dependency chain, which is done by connecting the **dstStageMask** to the **srcStageMask**. For example, we can make the dependency between vkCmdDispatch that need to transfer data from one to another.

```c++
1 vkCmdDispatch
2 vkCmdDispatch
3 vkCmdPipelineBarrier(srcStageMask = COMPUTE, dstStageMask = TRANSFER)
4 vkCmdMagicDummyTransferOperation
5 vkCmdPipelineBarrier(srcStageMask = TRANSFER, dstStageMask = COMPUTE)
6 vkCmdDispatch
7 vkCmdDispatch
```

In this example, 4 most wait for 1, 2 and 6, 7 wait for 4 to finish. Which is because the transfer is a dummy operation in this example we are achieving the result of ordering the commands by this memory barrier.
##### Access mask
These are used for memory barriers. Above here we have actually only been talking about execution barriers which is just concerned with the execution order of action commands. Memory barriers are also concerned with moving memory we need from the graphics cards L2 memory to L1 memory.

L2 memory on the GPU is considered available to but to be used by the warps, it needs to be moved to L1 memory to be considered visible. 

For example if we have an execution barrier that states that we need to wait for the `VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT` we might need to make sure that whatever data is available needs to become visible so we set the access mask to `VK_ACCESS_SHADER_READ_BIT`. 
##### How to make GPU memory available again
Lets say that in warp has done some written so colour values out after being processed. This would happen at the `VK_PIPELINE_STAGE_COLOUR_ATTACHMENT_OUTPUT_BIT` and we would have the `VK_ACCESS_COLOUR_ATTACHMENT_WRITE_BIT` mask to represent what type of memory we have in the L1 cache.

There is a BIG problem. Now the L1 cache in that warp isn't the same as the L2 cache meaning it's not available to be used by other warps. This is why we both use a source and destination access to mask to make sure the memory transfer is completed before we move on with further execution.

To fix this you need to combined both stage and access mask together like this.
```c++
VkMemoryBarrier2 memoryBarrier;
memoryBarrier.srcStageMask = VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT;
memoryBarrier.srcAccessMask = VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT;
```

In the example were the L1 cache needs to update the L2 cache we need to finish the `VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT` the `VK_ACCESS_COLOR_ATTACHMENT_WRITE_BIT` change needs to moved to the L2 cache making it visible. 

The destination access mask work with the destination stage mask to flag memory that needs to become visible for the destination stage before executing. 
```c++
memoryBarrier.dstStageMask = VK_PIPELINE_STAGE_COLOR_FRAGMENT_SHADER_BIT;
memoryBarrier.dstAccessMask = VK_ACCESS_COLOR_SHADER_READ_BIT;
```

For example, lets say we have `vkCmd1` that writes out colour memory to it's L1 cache. `vkCmd2` wants to do something after that `vkCmd1` has finish AND uses it's memory output in it's shader.

The source access moves the memory from `vkCmd1` L1 cache to L2, with the destination mask tells `vkCmd2` to transfer memory from L2 to it's L1 making it visible so it can complete it's operation in it's fragment shader stage.
##### Non-global memory barriers
When using synchronisation2 you can use `VkDependencyInfo` which restrict what the memory barrier/execution barrier operates under.

```c
typedef struct VkDependencyInfo {
    VkStructureType                  sType;
    const void*                      pNext;
    VkDependencyFlags                dependencyFlags;
    uint32_t                         memoryBarrierCount;
    const VkMemoryBarrier2*          pMemoryBarriers;
    uint32_t                         bufferMemoryBarrierCount;
    const VkBufferMemoryBarrier2*    pBufferMemoryBarriers;
    uint32_t                         imageMemoryBarrierCount;
    const VkImageMemoryBarrier2*     pImageMemoryBarriers;
} VkDependencyInfo;
```

Here you can see 3 different types of barriers you can set up. Image and buffer are just used to restrict between buffer based action commands and image based action commands. The **VkMemoryBarrier2** is used for global barriers.

This will increase performance as your setting up hard synchronisation barriers for every time of memory that doesn't require it.

It's important to note that render passes are just a very round about way for image memory barriers. As when you eventually finish working with render passes they output a VkImageView inside a VkFramebuffer that goes to the VkSwapchain.