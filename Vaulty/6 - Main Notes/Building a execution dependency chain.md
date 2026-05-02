2026-04-27 13:58
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# Building a execution dependency chain

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

In this example, 4 most wait for 1, 2 and 6, 7 wait for 4 to finish. Which is a transfer command in this example we are achieving the result of ordering the commands by this memory barrier.
# References
##### Main Notes
[[Synchronisation events aka split barriers]]
[[In-queue execution pipeline barriers]]
[[The pipeline stages]]
[[Execution pipeline barrier example]]
#### Source Notes
[[Vulkan synchronisation explained]]