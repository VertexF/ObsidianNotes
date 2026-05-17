2026-05-17 08:20
Status: #baby 
Tags: [[threading]]
# Introduction to mutex and locking

Shared data that we are concerned with is anything that has an **invariant** like pointers in a linked list that can be broken at run time, causing a bad race condition.

A mutex (**mut**ual **ex**clusion) is a primitive that tells threads that to wait before accessing shared data if another thread is accessing it. You **lock** the mutex when a thread access shared data, and you on **unlock** it once a thread has finished working on it. 

The standard library make sure that when a thread tries to lock a thread it can't, only when it's **unlocked** is that possible. This make sure that threads have self-consistent view on shared data.

You can't just 100% rely on these, you need to structure your interface to your code to avoid race conditions as well. You have to be careful you don't end up with a **deadlock** and protecting too much or too little.
# References
##### Main Notes
#### Source Notes
[[C++ Concurrency - Protected Shared Data]]