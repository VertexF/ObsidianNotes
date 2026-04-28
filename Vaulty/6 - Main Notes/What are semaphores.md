2025-11-07 13:33
Status: #baby 
Tags: [[vulkan]] [[vulkan sychronisation]]
# What are semaphores

Semaphores are used to order commands within a queue and synchronise different queues together. There is also functions outside of command synchronisation that need semaphores.

There are two types of semaphores **binary** and **timeline**. Timeline semphores are cored in 1.2 and needs an extension to get this working in early versions. Binary semaphores are the simplest and require less set up but can do less. 

A binary semaphore can be either unsignalled or signalled. It begins as unsignalled. This works by one synchronisation command in a queue signalling the semaphore when the command is complete, another command will wait on that semaphore until it is signalled to then go on and do it's work. Once the second command has started the semaphore goes back to unsignalled automatically.

Semaphores are different to barriers because they are interested in setting up barriers between group of action commands, rather than individual commands on the GPU.

So if you have a batch of draw command for example, that batch can wait on a semaphore before executing after submitted to **vkQueueSubmit**. As we are waiting for a batch of commands after a **vkQueueSubmit** we are waiting on the device and not the the host. A fence does the same thing but waits on the **host** side not on the device. So with a fence we are waiting for a signal to arrive after the batch of work has been completed on the device.

A perfect time to use a semaphore is when you have a batch of commands in queues that need to be executed after each other.

![[binarySemaphores.png]]

In this example queue 1 signals a semaphore queue 2 waits on. You can think about that in the above example that queue 1 is a graphics queue were all the work on the GPU is done and the second is a presentation queue were things needs to be presented to the screen.
# References
##### Main Notes
[[Implicit synchronisation guarantees]]
[[Vulkan command categories]]
[[Setting up the synchronisation objects]]
[[The theory  of synchronising a frame]]
#### Source Notes
[[Drawing a triangle]]
[[Vulkan synchronisation explained]]
