2026-05-16 15:34
Status: #baby 
Tags: [[threading]]
# Choosing the number of threads at runtime

In the threading library you have `std::thread::hardware_concurrency()` which is a helper function that hints on the number of threads the program can use. If it returns 0, then no information was able to be gathered.
# References
##### Main Notes
[[Introduction to multithreading]]
[[Introduction to how C++ threads works]]
[[Launching a thread]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]