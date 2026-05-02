2026-05-01 09:46
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# Implicit memory guarantees when waiting for a semaphore

Signalling a semaphore makes all memory available, waiting for a semaphore make memory visible. This basically means if you're ordering your synchronisation with semaphores you wont need memory barriers as it does the transition for you. If you have a pair of signal/wait they operate as full memory barriers.

For example, if you have one queue that is 1 writes to an SSBO in a compute shader and that data gets consummed in a UBO in a fragment shader in queue 2. You can set up semaphores between these queue if they are `QUEUE_FAMILY_CONCURRENT`

Queue 1 would look like this

```c++
vkCmdDispatch();
vkQueueSubmit(signal = mySemaphore);
```

Note here we don't need a pipeline barrier, signallng the semaphore wait for all commands and writes this dispatch memory to be made available before semaphore is actually signalled.

Queue 2 would look like this

```c++
vkCmdBeginRenderPass(); /*OR*/ vkCmdBeginRendering();
vkCmdDraw();
vkCmdEndRenderPass();/*OR*/ vkCmdEndRendering();
vkQueueSubmit(wait = mySemaphore, pDstWaitStageMask = FRAGMENT_SHADER);
```

When we wait for a semaphore, we specify wich stages should wait on this semaphore in our case the FRAGMENT_SHADER stage. The semaphore is takes care of moving the memory layout safely to what we need which would be UNIFORM_READ_BIT which is nice.
# References
##### Main Notes
[[What are semaphores]]
[[In-queue execution pipeline barriers]]
[[What are pipeline barriers]]
[[The pipeline stages]]
[[Implicit synchronisation guarantees]]
#### Source Notes
[[Vulkan synchronisation explained]]
