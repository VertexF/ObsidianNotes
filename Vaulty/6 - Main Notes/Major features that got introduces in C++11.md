2025-12-23 19:36
Status: #baby 
Tags: [[C++]]
# Major features that got introduces in C++11

`auto` got introduced and a good rule of thumb is to use it when you don't know what the type is.

Enchanced for loops, varadic templates, smart pointers also got added in this standard.

```c++
template<typename Func, typename ... T>
void doSomething(const Func& function, const T& ... param)
{
	function(param...);
}
```
# References
##### Main Notes
[[Major features that got introduces in C++14]]
[[Major features that got introduces in C++17]]
#### Source Notes
[[C++ Important Parts of C++11]]