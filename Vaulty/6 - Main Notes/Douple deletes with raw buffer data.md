2025-12-07 11:05
Status: #baby 
Tags: [[C++]]
# Douple deletes with raw buffer data

You need to be careful with handling raw data you are planning on allocating on the heap for things like pixel data or any other type because you can accidently lead to a double delete.

In this example here we are leaving things up the compiler to copy and move. Which goes wrong.
```c++
#include <stdint.h>
#include <iostream>
#include <cstring>

struct Image
{
	Image() 
	{
		pixelData = nullptr;
	}

    ~Image()
	{
		if (pixelData != nullptr)
		{
			delete[] pixelData;
		}
	}

	Image(uint32_t buffSize, uint8_t* data) : bufferSize(buffSize)
	{
		pixelData = new uint8_t[buffSize];

		memcpy(pixelData, data, bufferSize);
	}

	uint32_t bufferSize = 0;
	uint8_t *pixelData = nullptr;
};

struct Entity
{
	Image sprite[100];
} entities;


static Image loadSprint(const char* filePath) 
{
	//Load the file from file path
	uint32_t bufferSize = 8;
	uint8_t data[] = { 0xFF, 0x0F, 0xF0, 0xFF, 0x0F, 0xF0, 0xFF, 0x0F };

	return { bufferSize, data };
}

int main()
{
	int playerID = 0;
	entities.sprite[playerID] = loadSprint("../Sprint/frog.png");

	for (uint32_t i = 0; i < entities.sprite[playerID].bufferSize; ++i)
	{
		std::cout << "Players sprint colours are " << entities.sprite[playerID].pixelData[i] << "\n";
	}

	return 0;
}
```

What ends up happening is that the move operator that is generated from the compiler at this line. No RVO can save you here.

```c++
entities.sprite[playerID] = loadSprint("../Sprint/frog.png");
```

This move operator just the pointer of **pixelData** over to the `entities.sprite[playerID]` and then because **loadSprint** is an rvalue the the destructor gets called, meaning that the line `delete[] pixelData;` within the Image destructor runs and deletes pixelData from memory. Now `entities.sprite[playerID]` has junk data and when it runs it's own destructor the tries to delete already deleted memory at the end of main causing a double delete.

What you have to do is follow the rule of 5 and build out the constructs and operators to handle doing a deep copy so every case is taken care of.
```c++
#include <stdint.h>
#include <iostream>
#include <cstring>

struct Image
{
	Image() = default;
	Image(uint32_t buffSize, uint8_t* data) : bufferSize(buffSize)
	{
		pixelData = new uint8_t[buffSize];
		memcpy(pixelData, data, bufferSize);
	}

	Image(const Image& rhs)
	{
		bufferSize = rhs.bufferSize;
		pixelData = new uint8_t[bufferSize];
		memcpy(pixelData, rhs.pixelData, bufferSize);
	}

	Image(Image&& rhs) noexcept : 
		bufferSize(rhs.bufferSize),
		pixelData(rhs.pixelData)
	{
		rhs.bufferSize = 0;
		rhs.pixelData = nullptr;
	}

	Image& operator=(const Image& rhs)
	{
		if(&rhs == this)
	    {
			return *this;
		}

		delete[] pixelData;

		bufferSize = rhs.bufferSize;
		memcpy(pixelData, rhs.pixelData, bufferSize);

		return *this;
	}

	Image& operator=(Image&& rhs) noexcept
	{
		if (&rhs == this)
		{
			return *this;
		}

		delete[] pixelData;

		bufferSize = rhs.bufferSize;
		pixelData = rhs.pixelData;

		rhs.bufferSize = 0;
		rhs.pixelData = nullptr;

		return *this;
	}

    ~Image()
	{
		std::cout << "~Image()" << '\n';

		if (pixelData != nullptr)
		{
			delete[] pixelData;
		}
	}

	uint32_t bufferSize = 0;
	uint8_t *pixelData = nullptr;
};
```

The operators need to delete the old **pixelData** because the old data would still be lying around in memory. We would have a memory with no way to access it.

The move constructors and operators don't need to have a memory copy because the we have a r-reference object to deal with so the data can be safely moved without a **memcpy**. We do want to 0 out the memory after the move because **.rhs.pixelData** after the move will have junk in.

You get a warning if you need add **noexecpt** in the move operator and constructor. Remember this just means that if an expection is thrown in a function marked as **noexpect** the application just terminates 
# References
##### Main Notes
#### Source Notes
