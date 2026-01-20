2025-11-04 08:37
Status: #baby 
Tags: [[vulkan]] [[vulkan command buffers]]
# What are command buffers

Command buffer are objects that you record and later run commands on within a render pass. Recording commands in bulk allow vulkan to efficiently run the commands. Recording command buffer is thread friendly meaning you can record things on different threads.

Commands run by submitting commands buffers every frame to a device queue such as a presentation queue or a graphics queue. Submision to queues is not thread friendly.
# References
##### Main Notes
[[Setting up the command pool]]
[[Setting up the command buffers]]
[[What is a render pass]]
#### Source Notes
[[Drawing a triangle]]
