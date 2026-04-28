2026-04-27 12:17
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# Implicit synchronisation guarantees

When talking about synchronisation between commands it's important to note that vulkan has implicit synchronisation guarantees. This means that command recorded to a command buffer AND submission to a `vkQueueSubmit`ordering matters.
# References
##### Main Notes
[[What are fences]]
[[What are semaphores]]
[[Vulkan command categories]]
[[Vulkan command categories]]
[[The theory  of synchronising a frame]]
#### Source Notes
[[Vulkan synchronisation explained]]