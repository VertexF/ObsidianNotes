2026-04-27 13:37
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# The difference between source and destination stage mask

VkPipelineStageFlags **srcStageMask** and **dstStageMask** can be thought of one being the "consumer" stage (**srcStageMask**) and "producer" stage (**dstStageMask**). By specifying the source and target stages you tell the driver what operations need to finish before the transition can execute (the **srcStageMask**) and what can't have started yet (the **dstStageMask**).

For example if you have this memory barrier set up like this you have.

```c++
vkCmdPipelineBarrier(
    commandBuffer,
    VK_PIPELINE_STAGE_BOTTOM_OF_PIPE_BIT, // source stage
    VK_PIPELINE_STAGE_TOP_OF_PIPE_BIT,    // destination stage
    /* remaining parameters omitted */);
```

When you add this memory barrier you are saying that every action command needs to finish and no work must start before the transition happens. Effectively the barrier will wait for everything to finish and block work from starting. This isn't very good because you have a pipeline bubble.

![[top_of_pipe_and_bottom_of_pipe_barrier.png]]

Let use a better example lets say you want a vertex shader that stores data via and`imageStore` and you want that to be consumed by a compute shader. In this case wouldn't want to wait for the fragment shader to finish so we let piece of work pass through. If there is a computer shader command we must wait for the vertex shader to finish. 

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

So in the first command we are skipping over the compute shader part of the pipeline because the command does a noop on that pipeline stage so nothing is needed to be waited on.
##### IMPORTANT
This is extremely important that you don't get recording commands or submitting a queue mixed up with work being done which reference to an action running through it's pipeline stages to execute. 
# References
##### Main Notes
[[In-queue execution pipeline barriers]]
[[The pipeline stages]]
[[Execution pipeline barrier example]]
#### Source Notes
[[Vulkan synchronisation explained]]
