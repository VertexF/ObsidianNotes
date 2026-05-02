2026-04-27 12:20
Status: #baby 
Tags: [[vulkan]] [[vulkan queue]] [[vulkan synchronisation]]
# Basic queue synchronisation

###### The Vulkan queue
Synchronisation of queues in vulkan is basically the same as 1 queue. A queue is just an abstraction of vulkan blasting through commands. 
##### Command buffer misconceptions
All commands are global to a queue so synchronisation to anything within that queue is global to all commands.
# References
##### Main Notes
[[Vulkan command overlap]]
[[Implicit synchronisation guarantees]]
#### Source Notes
[[Vulkan synchronisation explained]]