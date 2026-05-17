2026-05-16 11:07
Status: #baby 
Tags: [[threading]]
# Passing argument basics with threads

When pass an argument into standard thread with `std::threads(functionName, arg1, arg2);` which matches the parameters of the function, it's all pass by copy into the internal data within `std::thread` these then get passed back out to the function as **r-values** as if they are temporaries. 
# References
##### Main Notes
[[Thread string arguments]]
[[Non-const reference arguments with threads]]
[[Member function arguments with threads]]
[[The standard thread ownership rule]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]
