2026-04-29 12:06
Status: #baby 
Tags: [[vulkan]] [[vulkan synchronisation]]
# No TOP_OF_PIPE and BOTTOM_OF_PIPE access masks

Doing this is completely meaningless. The `TOP_OF_PIPE/BOTTOM_OF_PIPE` are only for execution barriers which are strictly different than memory barriers. The reason is that these stages perform no accesses to memory when they are completed.
# References
##### Main Notes
[[What are the access masks]]
[[How GPU memory works]]
#### Source Notes
[[vulkan synchronisation explained]]
