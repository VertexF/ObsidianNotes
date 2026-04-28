2025-11-07 13:41
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# What are fences

Fences are similar to wait idle operations be allow finer control over how we are waiting. This starts like anything else with a **vkQueueSubmit** command then the device starts working through the batch of commands and uses an appropriate queue to handle these commands.

Fences are signalled after a batch of work has finished working. Meaning that it's important to consider the batches you're sending the the **vkQueueSubmit**. So if we have a queue with more than 1 batch, the fence signal state is dependent **MOSTLY** on those commands in that batch.

The typically case is if you have **cmd1**, **cmd2**, **cmd3** in a batch and **cmdx** which isn't in the batch that runs **AFTER** the batch of commands. Even if **cmdx** isn't finished the fence will be signalled if **cmd1**, **cmd2**, **cmd3**. 

It's important to note there is an exception to this fence batch rule, if you have a command that isn't in the queue but runs before the **BEFORE** the batch **AND** the command takes longer than all the commands in the queue. The fence will be signalled when the slow command has finished.

This is what the spec says about this *`Fence signal operations that are degined by vkQueueSubmit additionally include the first synchronisation scrope all commands that occur earlier in submission order.`* When we talk about **first synchronisation scope** we are talking about all the commands before the fence in the queue. The only thing in the **second synchronisation scope** is the fence operation itself and builds up the **execution dependency**.

The two scopes build up an **execution dependency** between command batches. Which basically states that the first scope most finish before the second. 

The scopes aren't exclusive to fences. First and second synchronisation scope and execution dependencies are how vulkan works with everything else.

If you can avoid fences because the CPU will just wait until it gets a signal from the fence which isn't useful work. You also need to reset fences manually. 
# References
##### Main Notes
[[Implicit synchronisation guarantees]]
[[Vulkan command categories]]
[[The theory  of synchronising a frame]]
[[What are wait idle operations]]
[[What are semaphores]]
[[Setting up the synchronisation objects]]
#### Source Notes
[[Drawing a triangle]]
[[Vulkan synchronisation explained]]