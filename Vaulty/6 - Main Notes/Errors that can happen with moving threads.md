2026-05-16 14:32
Status: #baby 
Tags: [[threading]]
# Errors that can happen with moving threads

Transferring ownership might be useful if you want a thread to run in the back and pass ownership to a new thread back to the calling function rather than waiting for it to complete.

Or you might want to create a thread and pass ownership in to some function that should wait for it complete. In either case you need to worry about how to move these threads around.

This is an example of things that might go wrong.

```c++
void foo(){}
void bar(){}
std::thread t1(foo);
std::thread t2 = std::move(t1);

t1 = std::thread(bar);
std::thread t3;
t3 = std::move(t2);
t1 = std::move(t3); //This move will terminate the program!!
```

After all these moves, `t1` is associated with the thread running `bar()`, `t2` has no associated thread, and `t3` is associated with the thread running `foo()`. 

The final move put back `t1` with it's original function, but do the moves it already had ownership with the function `bar()`. This will call the dreaded function `std::terminate`.

This is because you can't just drop a `std::thread` by assigning a new function to it. It still needs to be joined or detached.
# References
##### Main Notes
[[Moving threads with functions]]
[[The standard thread ownership rule]]
[[Using moves to safe guard against thread life time mistakes]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]