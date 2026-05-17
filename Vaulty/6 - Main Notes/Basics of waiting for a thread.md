2026-05-16 07:57
Status: #baby 
Tags: [[threading]]
# Basics of waiting for a thread

If you need to wait for a `std::thread` to finish you just the `.join` function call on the thread object. Joining the thread is still even necessary even if the thread has finished with it's work. This is a heavy weight synchronisation function call, as it waits for the entire thread to finish. If you want finer control you need **conditional variable** and **futures**.

Calling `join()` also cleans up any storage that's tied to that thread. Meaning you can't call `join()` again and `joinable()` will return false.
# References
##### Main Notes
[[Potential issues with joining a thread]]
[[Launching a thread]]
[[Potential issues with detaching a thread]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]