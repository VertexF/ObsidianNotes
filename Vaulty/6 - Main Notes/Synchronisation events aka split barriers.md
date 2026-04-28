2026-04-27 14:00
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# Synchronisation events aka split barriers

With events you can have a barrier that overlap work. You can use **VkEvents** to split up a memory barrier. The idea here is that the command inbetween that can overlap with other command before or after the set of commands.

For example if I have a `vkCmd1` and `vkCmd2` which have dependency on each other. However while `vkCmd2` waits for `vkCmd1` to finish, we could have `vkCmdA` and `vkCmdB` while while the pause is happening.

```c++
vkCmd1;
vkCmdSetEvent(X, VK_PIPELINE_STAGE_COLOR_ATTACHMENT_OUTPUT_BIT);
vkCmdA;
vkCmdB;
vkCmdSetEvent(X, VK_PIPELINE_STAGE_COLOR_FRAGMENT_SHADER_BIT);
vkCmd2;
```

Now `vkCmdA` and `vkCmdB` run whenever they like. 

Not all GPUs take advantage of this feature but it can be very important for advanced compute work. The reason being if there is a gap that the barrier you can through something else that fills up the GPU with work.
# References
##### Main Notes
[[Building a execution dependency chain]]
[[The difference between source and destination stage mask]]
[[The pipeline stages]]
[[Vulkan command categories]]
[[Implicit synchronisation guarantees]]
[[Basic queue synchronisation]]
[[Vulkan command overlap]]
#### Source Notes
[[Vulkan synchronisation explained]]