2026-05-16 11:16
Status: #baby 
Tags: [[threading]]
# The standard thread ownership rule

Unique pointers have explicit ownership rules which is the same as `std::thread` it also has same rules. As `std::thread` can be **moved** and NOT copied. This allow programmer to transfer ownership between object while making sure there is 1 `std::thread` for 1 thread.
# References
##### Main Notes
[[Errors that can happen with moving threads]]
[[Arguments that can only be moved with threads]]
[[Passing argument basics with threads]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]