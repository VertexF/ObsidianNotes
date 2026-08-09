2026-07-27 13:36
Status: #baby 
Tags: [[C++]]
# What are functors

These aren't functions please never confuse them. Functors are object that can treated as though thye are functions or functions points. Functors are very common in the STL.

Lets state problem they can solve with this code snippet
```c++
#include <algorithm>
#include <stdio.h>

int increment(int x) { return x + 1; }

int main()
{
	int arr[] = { 1, 2, 3, 4, 5 };
	int sizeOfArray = sizeof(arr) / sizeof(arr[0]);
	
	//Apply the increment to all of the element of the arr and return the modified arr in the third arg.
	std::transform(arr, arr + sizeOfArray, arr, increment);
	
	for(uint32_t i = 0; i < sizeOfArray; ++i)
	{
		printf("%d ", arr[i]);
	}
	
	return 0;
}
```

Now suppose we wanted to add `5` to each element or `10`, `-2` If we were to use `std::transform` we would be in bad situation as we need countless different functions. We can't pass in arguments to `increment()` either because that's used internally within `std::transform`

This brings in the **functor** (function object) which is a C++ class taht acts like a function. **Functors** are called using the same old function syntax. To create a functor we will need using the operator overload ().

```c++
//The functor
class Increment
{
public:
	Increment(int n) : num(n)
	{
	}

	//This enables the calling of the operator function () on an object of Increment.
	int operator()(int arrayNum) const
	{
		return num + arrayNum;
	}
private:
	int num;
};

int main()
{
	int arr[] = { 1, 2, 3, 4, 5 };
	int sizeOfArray = sizeof(arr) / sizeof(arr[0]);

	//Apply the increment to all of the element of the arr and return the modified arr in the third arg.
	std::transform(arr, arr + sizeOfArray, arr, Increment{5});

	for (uint32_t i = 0; i < sizeOfArray; ++i)
	{
		printf("%d ", arr[i]);
	}

	return 0;
}
```

The line `Increment{5}` is triggering the object with the argument of `5` to be passed into the constructor. Then the sum `num + arrayNum;` in the functor basically does a `5 + 1, 5 + 2, ... 5 + 5,` adding `5` to all of them instead of 1.

If you wanted you can make an object as an r-value and pass in that instances into `std::transform` instead of directly as an l-value.
# References
##### Main Notes
#### Source Notes
