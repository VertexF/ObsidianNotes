2026-05-16 11:14
Status: #baby 
Tags: [[threading]]
# Arguments that can only be moved with threads

This is actually pretty straight forward if we consider a `std::unique_ptr` as it can't be copied, you can't pass this into the `std::thread` constructor unless you explicitly use the `std::move` function to do it.

```c++
void process(std::unique_ptr<obj>)
{
}
std::unique_ptr<obj> p = std::make_unique();
p->prepareData(43);
std::thread f(process, std::move(p));
```

Now the ownership of the unique_ptr is transferred over to the internals of `std::thread`.
# References
##### Main Notes
[[The standard thread ownership rule]]
[[Non-const reference arguments with threads]]
[[Member function arguments with threads]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]