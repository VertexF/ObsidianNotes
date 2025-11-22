2025-11-07 14:03
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# Using a fence to synchronise the previous frame

At the beginning of the drawing routine you'll need to wait for the previous frame.  You do this with the **vkWaitForFence**

```c++
vkWaitForFences(device, 1, &framesInFlight[currentFrame], VK_TRUE, UINT64_MAX);
```

The **vkWaitForFence** can wait more muiltple fences. The secound argument is the amount of fences in the array, the next is the VkFence array and the **VK_TRUE** here is a bool that says should the host wait for all the fences. Since we are using 1 fence this doesn't matter to use a lot. The last argument is the timeout, using UINT64_MAX pretty much disables this wait timing out. 

Remember you have to reset the fence manually so after the wait is over from **vkWaitForFences** we need to reset the fence back it's unsignalled state with.

```c++
vkResetFences(device, 1, &framesInFlight[currentFrame]);
```
# References
##### Main Notes
[[What are fences]]
[[Setting up the synchronisation objects]]
[[Handling frames in flight]]
#### Source Notes
[[Drawing a triangle]]