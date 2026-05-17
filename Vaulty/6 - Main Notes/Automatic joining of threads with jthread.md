2026-05-16 14:38
Status: #baby 
Tags: [[threading]]
# Automatic joining of threads with jthread

Now wouldn't it be nice if you had a thread that would automatically do this without the need for use to [[Using moves to safe guard against thread life time mistakes|write our own class?]]

Welcome the C++20 `std::jthread` which follows the same rules as `std::thread` but it joins automatically on destruction. You can detach, swap ownership with other `std::jthread` object just like `std::thread`, but you are safely guarded against forgetting to join at the end of the `std::thread` life time.
# References
##### Main Notes
[[Using moves to safe guard against thread life time mistakes]]
[[The standard thread ownership rule]]
[[Errors that can happen with moving threads]]
[[Moving threads with functions]]
[[The standard thread ownership rule]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]