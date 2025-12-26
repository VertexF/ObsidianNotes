2025-12-23 19:41
Status: #baby 
Tags: [[C++]]
# Major features that got introduces in C++17

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
# References
##### Main Notes
[[Major features that got introduces in C++11]]
[[Major features that got introduces in C++14]]
#### Source Notes
[[C++ Important Parts of C++11]]