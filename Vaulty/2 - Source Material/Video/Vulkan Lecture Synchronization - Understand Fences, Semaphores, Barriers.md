# Reference https://www.youtube.com/watch?v=GiKbGWI4M-Y

It's important to know that in vulkan that submission order of these commands is meaningful. Meaning that if you submit **cmd1**, then **cmd2** vulkan will start the commands in that order, but they could finish in any order. This needs to be considered carefully when it comes to synchronisation.

You have different categories of commands. 

>**Action commands**
 Graphics Pipeline Commands:  
>>vkCmdDraw
>>vkCmdIndexed
>>vkCmdIndirect
>>vkCmdDrawMeshTasksNV
>>vkCmdClearAttachments
 Compute Pipeline Commands:
>>vkCmdDispatch
>>vkCmdDispatchIndirect
>>Ray Tracing Pipeline Commands:
>>vkCmdTraceRaysKHR
>>vkCmdTraceRaysIndirectKHR
 Transfer Commands:
>>vkCmdCopyBuffer
>>vkCmdCopyImage
>>vkCmdCopyBufferToImage
>>vkCmdCopyImageToBuffer
>>vkCmdCopyAccelerationStructureKHR
>>vkCmdBlitImage
>>vkCmdResolveImage
>>vkCmdClearColorImage

Each of the subcategories in this list use different queues to get the work done. 

>**Sychronisation commands**
>>vkCmdPipelineBarrier/2
>>vkCmdSetEvents/2
>>vkCmdWaitEvents/2
>>vkCmdBeginRenderPass/2
>>vkCmdEndRenderPass/2

>**State commands**
 Bind Commands:
>>vkCmdBindDescriptorSets
>>vkCmdBindPipeline
>>vkCmdBindVertexBuffers
>>vkCmdBindIndexBuffer
 Other Commands:
>>vkCmdPushConstants
>>vkCmdPushDescriptorSetKHR
>>vkCmdSetScissor
>>vkCmdSetViewport
>>vkCmdSetDepthBias

It's possible for example the fragment shader needs to wait for a **vkCmdCopyBufferToImage** needs to finish before it can do anything. This is what synchronisation is needed for. It's possible for example the **vkCmdCopyBufferToImage** to write directly to a **VkImage** which points to some buffer in **VkMemory**. These happen in a command batch, in a command queue. It's possible for example we set the **vkDraw** command up so it starts after **vkCmdCopyBufferToImage** but the copy command is slow so it doesn't finish before the draw command is submitted, now you have a problem.
##### Wait idle operations
These are the heaviest of all synchronisation method, wait idle operations. This starts like anything else with a **vkQueueSubmit** command then the device starts working through the batch of commands and uses an appropriate queue to handle these commands. Once the commands have finished the queue goes to an **idle** state.

We can wait on a queue going idle with 
```c++
VkQueue queue;
vkQueueWaitIdle(queue);
```

You can also wait on the entire device with a similar command, but now you're waiting for all work to finish on the device before moving on.

```C++
VkDevice device;
vkDeviceWaitIdle(device);
```

These are used to wait on the host side for device work to finish. 
##### Fence operations
Fences are similar to wait idle operations be allow finer control over how we are waiting. This starts like anything else with a **vkQueueSubmit** command then the device starts working through the batch of commands and uses an appropriate queue to handle these commands.

Fences are signalled after a batch of work has finished working. Meaning that it's important to consider the batches you're sending the the **vkQueueSubmit**. So if we have a queue with more than 1 batch, the fence signal state is dependent **MOSTLY** on those commands in that batch.

The typically case is if you have **cmd1**, **cmd2**, **cmd3** in a batch and **cmdx** which isn't in the batch that runs **AFTER** the batch of commands. Even if **cmdx** isn't finished the fence will be signalled if **cmd1**, **cmd2**, **cmd3**. 

It's important to note there is an exception to this fence batch rule, if you have a command that isn't in the queue but runs before the **BEFORE** the batch **AND** the command takes longer than all the commands in the queue. The fence will be signalled when the slow command has finished.

This is what the spec says about this *`Fence signal operations that are degined by vkQueueSubmit additionally include the first synchronisation scrope all commands that occur earlier in submission order.`* When we talk about **first synchronisation scope** we are talking about all the commands before the fence in the queue. The only thing in the **second synchronisation scope** is the fence operation itself and builds up the **execution dependency**.

The two scopes build up an **execution dependency** between command batches. Which basically states that the first scope most finish before the second. 

The scopes aren't exclusive to fences. First and second synchronisation scope and execution dependencies are how vulkan works with everything else.
##### Semaphore
A semaphore is used to synchronise things between batches of commands. So if I say have three commands in a batch I can set up a semaphore before and after these commands making sure they are synchronised. 

The difference between a **fence** and a **semaphore** is where the waiting for a signal can happen. A semaphore can be waited on by another batch of work, that is work recorded into a command and submitted to a queue. So if you have a batch of draw command for example, that batch can wait on a semaphore before executing after submitted to **vkQueueSubmit**. As we are waiting for a batch of commands after a **vkQueueSubmit** we are waiting on the device and not the the host. A fence does the same thing but waits on the **host** side not on the device. So with a fence we are waiting for a signal to arrive after the batch of work has been completed on the device.

A perfect time to use a semaphore is when you have a batch of commands in queues that need to be executed after each other.

![[binarySemaphores.png]]

In this example queue 1 signals a semaphore queue 2 waits on. You can also image thing picture above as different frame if each queue has all the work needed to display a frame.
##### Pipeline barriers
These here to make sure thinks don't access other commands resources before a previous command has finished. For example if I have a **vkCmdCopyBufferToImage** that's going to write to a **VkImage** that the fragment shader needs to read from, I can submit these in order as you would expect but it's possible that **vkCmdCopyBufferToImage** takes a long time, so it's possible even if it's submitted first that the fragment shader will access the **VkImage** before it's ready. So here you'll need to set up a pipeline barrier between commands copy and whatever runs the fragment shader.