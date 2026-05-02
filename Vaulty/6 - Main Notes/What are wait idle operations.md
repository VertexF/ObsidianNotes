2026-04-27 12:25
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# What are wait idle operations

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
# References
##### Main Notes
[[Vulkan command categories]]
[[Implicit synchronisation guarantees]]
[[Basic queue synchronisation]]
[[Vulkan command overlap]]
[[What are fences]]
#### Source Notes
[[Vulkan synchronisation explained]]