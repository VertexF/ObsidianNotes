2025-11-30 10:11
Status: #baby 
Tags: [[C++]]
# Dangling pointer with locally created arrays

Look at the code for explanation.

```c++
#include <stdint.h>
#include <iostream>

struct A 
{
	char sType;
	const void* pNext;
};

struct B 
{
	char sType;
	const void* pNext;
};

int main()
{
	A apple{};
	apple.sType = 'A';
	uint32_t arraySize = 1;

	B blue{};
	if (1)
	{
		A apple2{};
		apple2.sType = 'C';

		arraySize++;
		
		//This is a locally stored array that's on the stack.
		A ar[] = {apple, apple2};

		blue.sType = 'B';
		//The .pNext pointer points to this array.
		blue.pNext = &ar;
		
		//After that, we reach the end of the if block.
		//Now `ar` is popped of the stack.
		//.pNext is now a dangling pointer, aka pointing to a piece of memory thas has been removed.
	}
	else 
	{
		blue.sType = 'B';
		blue.pNext = &apple;
	}

	//This is now UB, we are deferencing/casting a pointer that is pointing to memory that has been popped of the stack.
	const A* finalapple = (const A*)blue.pNext;
	for (uint32_t i = 0; i < arraySize; ++i)
	{
		std::cout << "The sType of the struct is : " << (finalapple + i)->sType << std::endl;
	}

	std::cout << arraySize << std::endl;

	return 0;
}
```

 Note that this code above works in debug mode even though it's UB. This is because the stack frame is much larger meaning that the memory can still exist even when we have a dangling pointer.

The code fixes things for us.
```c++
#include <stdint.h>
#include <iostream>

struct A 
{
	char sType;
	const void* pNext;
};

struct B 
{
	char sType;
	const void* pNext;
};

int main()
{
	A apple{};
	apple.sType = 'A';
	uint32_t arraySize = 1;

	if (1)
	{
		B blue{};
		A apple2{};
		apple2.sType = 'C';

		arraySize++;
		
		//We can still use this local array but we have to be aware that it gets destroyed at the end of the of the if block.
		A ar[] = {apple, apple2};

		blue.sType = 'B';
		blue.pNext = &ar;

		//`ar` still exists so the isn't UB anymore it correctly.
		const A* finalapple = (const A*)blue.pNext;

		//Now we do the work with the memory of `ar`. 
		for (uint32_t i = 0; i < arraySize; ++i)
		{
			std::cout << "The sType of the struct is : " << (finalapple + i)->sType << std::endl;
		}

		std::cout << arraySize << std::endl;
		//`ar` is destroyed but this doesn't matter because of the fact we have already used and wont use it again.
	}
	else 
	{
		B blue{};
		blue.sType = 'B';
		blue.pNext = &apple;

		const A* finalapple = (const A*)blue.pNext;

		for (uint32_t i = 0; i < arraySize; ++i)
		{
			std::cout << "The sType of the struct is : " << (finalapple + i)->sType << std::endl;
		}

		std::cout << arraySize << std::endl;

	}
	return 0;
}
```
# References
##### Main Notes
#### Source Notes
