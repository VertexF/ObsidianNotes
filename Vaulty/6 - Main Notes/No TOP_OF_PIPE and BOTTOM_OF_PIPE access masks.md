2026-04-29 12:06
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# No TOP_OF_PIPE and BOTTOM_OF_PIPE access masks

Doing this is completely meaningless. The `TOP_OF_PIPE/BOTTOM_OF_PIPE` are only for execution barriers which are strictly different than memory barriers. The reason that they are useless for memory barrier is that they stages perform no memory accesses when they are completed.

For example `TOP_OF_PIPE` pipeline stage is the stage is when the GPU parses the commands before execution. That doesn't tough memory we are interested in.
# References
##### Main Notes
[[What are the access masks]]
[[How GPU memory works]]
#### Source Notes
[[vulkan synchronisation explained]]
