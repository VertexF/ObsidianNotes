2026-04-27 13:52
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# What are the access masks

These are used for memory barriers. Above here we have actually only been talking about execution barriers which is just concerned with the execution order of action commands. Memory barriers are also concerned with moving memory we need from the graphics cards L2 memory to L1 memory.

L2 memory on the GPU is considered available to but to be used by the warps, it needs to be moved to L1 memory to be considered visible. 

For example if we have an execution barrier that states that we need to wait for the `VK_PIPELINE_STAGE_FRAGMENT_SHADER_BIT` we might need to make sure that whatever data is available needs to become visible so we set the access mask to `VK_ACCESS_SHADER_READ_BIT`. 
# References
##### Main Notes
[[How to make GPU memory visible and available]]
[[The difference between source and destination stage mask]]
[[Making non-global pipeline barriers]]
[[In-queue execution pipeline barriers]]
[[The pipeline stages]]
[[Vulkan command categories]]
[[Implicit synchronisation guarantees]]
[[Basic queue synchronisation]]
[[Vulkan command overlap]]
#### Source Notes
[[Vulkan synchronisation explained]]