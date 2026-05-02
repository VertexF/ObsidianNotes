2026-04-27 12:23
Status: #baby
Tags: [[vulkan]] [[vulkan command buffers]] [[vulkan synchronisation]]
# Vulkan command overlap

When you're recording command to a command buffer, the order is important and preserved because of [[Implicit synchronisation guarantees]]. The execution of commands in a queue is just a stream of commands from start to finish. Command will fire off one after another in sequence **BUT** end out of order.

This means that the actual execution of the commands is out of order, because starting a command isn't the same an executed command.

The graphics driver may overlap command like vkDraw and vkDispatch action commands. This is were you would might need a barrier if the vkDispatch is dependent on vkDraw, but remember all action commands will finish executing at a unspecified time meaning that you'll like need memory barriers anyway.

Framebuffer operations inside a render pass happen in order which is an exception.
# References
##### Main Notes
[[Implicit synchronisation guarantees]]
[[Basic queue synchronisation]]
#### Source Notes
[[Vulkan synchronisation explained]]