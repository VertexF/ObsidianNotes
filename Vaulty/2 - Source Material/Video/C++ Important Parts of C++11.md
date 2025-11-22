# Reference https://www.youtube.com/watch?v=D5n6xMUKU3A

`auto` got introduced and a good rule of thumb is to use it when you don't know what the type is.

Enchanced for loops, varadic templates, smart pointers also got added in this standard.

```c++
template<typename Func, typename ... T>
void doSomething(const Func& function, const T& ... param)
{
	function(param...);
}
```

##### C++14

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
##### C++17

One cool feature is `std::string_view` which has it's own header but does get included as part of `#include <string>`. This is for strings that aren't going to be changed, think of it as a read-only string. 

You also have class template argument deduction. Which instead of passing the values within a template C++ will work stuff out with it's arguments like
```c++
#include <array>
//C++14 and below
std::array<int, 3> data{ 1,2,3 };
//C++17 and above
std::array moreData{ 1,2,3 };
```

These can fall over so be careful.

You can also extract out values from a struct, class or pair and name them to variables, with some new syntax.

```c++
struct blob 
{
	int x;
	float y;
	char z;
};

void foo() 
{
	blob b{22, 78.6, 'A'};
	auto [number, decimal, character] = b;
	std::cout << decimal << std::endl;
}
```

You can also initial variables within an if-statement. This is called if-init expressions. The variable is then used throughout the scope of the if-statement including the else and if else parts.

```c++
if (blob bob = {22, 0.5, 'Z'}; bob.x < 3)
{
	std::cout << bob.z << std::endl;
}
else if (bob.y == 0.5)
{
	auto lad = bob.y * 3;
	std::cout << lad << std::endl;
}
```