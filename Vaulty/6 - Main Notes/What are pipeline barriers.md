2026-04-27 12:31
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# What are pipeline barriers

The whole point of these is to stop overlapping of resources and/or memory from action commands. You generally have two types of pipeline barriers, memory and execution. The difference between is that execution **ONLY** cares about the order of action commands in a queue, memory cares about both memory AND execution order.  
# References
##### Main Notes
[[The pipeline stages]]
[[Vulkan command categories]]
[[Implicit synchronisation guarantees]]
[[Basic queue synchronisation]]
[[Vulkan command overlap]]
#### Source Notes
[[Vulkan synchronisation explained]]