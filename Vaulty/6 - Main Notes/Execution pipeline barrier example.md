2026-04-27 13:50
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# Execution pipeline barrier example

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
# References
##### Main Notes
[[The difference between source and destination stage mask]]
[[In-queue execution pipeline barriers]]
[[The pipeline stages]]
#### Source Notes
[[Vulkan synchronisation explained]]