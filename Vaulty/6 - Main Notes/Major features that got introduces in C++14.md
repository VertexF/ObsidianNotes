2025-12-23 19:38
Status: #baby 
Tags: [[C++]]
# Major features that got introduces in C++14

One of the coolest changes is with `constexpr` you can now use this in the context of loop, local variables, and branches. The only restriction you have with constexpr is that you can't call other functions that are not constexpr.

```c++
constexpr int foo() 
{
    int x = 9;
    if(x > 0)
        return 1;
    else
        return 99;
}

constexpr int fail()
{
    std::vector<int> blue{0, 1, 2, 3, 4, 5, 6};
    return 0;
}

int main()
{
    std::array<int, foo()> red;
    return 0;
}
```

`foo()` works completely and can be used as a template N parameter. Were as `fail` doesn't compile because it's allocating stuff on the heap which needs to happend at run time, so this wont work.

In lambdas you can now assign values within the capture clause, and this is crazily flexable, you can even return functions within the capture clause for example as an assignment.

```c++
auto lammy = [value = 9](auto i)
{
	if(i > value)
	{
		return "Fart";
	}
};
lammy(7);
```

You have `auto` in the paramters which is new. The `value = 9` type is worked out by the assignment itself. So you can't say `[float value = 7.f]` that wont compile. It's just `[value = 7.f]` for a float type for example.
# References
##### Main Notes
[[Major features that got introduces in C++11]]
[[Major features that got introduces in C++17]]
#### Source Notes
[[C++ Important Parts of C++11]]