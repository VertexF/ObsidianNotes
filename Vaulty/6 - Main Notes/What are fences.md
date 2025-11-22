2025-11-07 13:41
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# What are fences

Fences are used to synchronise between the host (CPU) and the GPU. If host needs to know when the GPU is done on the host we use fences.

Fences work similarly to binary semaphores. They are either signalled or unsignalled. When work is finished with a fence we can signal it which means the host can continue on because we know the GPU is up to date with the host now.

If you can avoid fences because the CPU will just wait until it gets a signal from the fence which isn't useful work. You also need to reset fences manually. 
# References
##### Main Notes
[[Setting up the synchronisation objects]]
[[What is synchronisation in vulkan]]
#### Source Notes
[[Drawing a triangle]]