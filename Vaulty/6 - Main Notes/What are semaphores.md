2025-11-07 13:33
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# What are semaphores

Semaphores are used to order commands within a queue and synchronise different queues together. There is also functions outside of command synchronisation that need semaphores.

There are two types of semaphores **binary** and **timeline**. To use timeline you need an extension to get this working. Binary semaphores are the simpliest and require less set up but can do less. Here I will only talk about binary semaphores as they are simplest.

A binary semaphore is either unsignalled or signalled. It begins as unsignaled. This works by one command in a queue signalling the semaphore when the command is complete, another command will wait on that semaphore until it is singled to then go on and do it's work. Once the second command has started the semaphore goes back to unsignaled automatically.

These are the sets a binary semaphore would go through to synchronise something:
1) Semaphore starts unsignalled
2) Command one begins running and will signal **AFTER** it's done.
3) Command two waits for that signal in the semaphore.
4) Command two runs when the signal is received.
5) Semaphore automatically goes back to unsignalled after the second command **STARTS**.

With the semaphores the synchronisation is **ONLY** for commands running that run on the GPU. The CPU is free to continue on while semaphores are doing their thing.
# References
##### Main Notes
[[Setting up the synchronisation objects]]
[[What is synchronisation in vulkan]]
#### Source Notes
[[Vulkan-Tutorial]]
