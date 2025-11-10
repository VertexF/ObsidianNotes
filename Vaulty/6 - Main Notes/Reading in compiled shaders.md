2025-10-19 18:39
Status: #baby 
Tags: [[vulkan]] [[vulkan shaders]]
# Reading in compiled shaders

First thing you need to know is that you need both the file size and the file in binary because this is in byte code. 

```c++
static long getFileSize(FILE* file)
{
	long fileSizeSigned;

	fseek(file, 0, SEEK_END);
	fileSizeSigned = ftell(file);
	fseek(file, 0, SEEK_SET);

	return fileSizeSigned;
}

//Read file and allocate memory from. User is responsible for freeing the memory.
static char* fileReadBinary(const char* filename, size_t* size = nullptr)
{
	char* outData = 0;

	FILE* file = fopen(filename, "rb");
	if (file)
	{
		size_t filesize = getFileSize(file);

		outData = (char*)malloc(filesize + 1);

		if (outData != nullptr) 
		{
			fread(outData, filesize, 1, file);
			outData[filesize] = 0;
		}
		else 
		{
			printf("Couldn't allocate memory for file %s", filename);
		}

		if (size)
		{
			*size = filesize;
		}

		fclose(file);
	}
	else 
	{
		printf("Couldn't open file %s", filename);
	}

	return outData;
}
```

I think the code pretty much explains itself here. 

This is how you use these function. In vulkan the shader binary code needs to be in **const uint32_t** so you have to reinterpet_cast<> to that value once you've read in the shaders.

```c++
size_t sizeOfVertexCode = 0;
size_t sizeOfFragmentCode = 0;

const uint32_t* binVertCode = reinterpret_cast<const uint32_t*>(fileReadBinary("Shaders/mainShader.vert.spv", &sizeOfVertexCode));

const uint32_t* binFragCode = reinterpret_cast<const uint32_t*>(fileReadBinary("Shaders/mainShader.frag.spv", &sizeOfFragmentCode));
```

**WARNING** If you haven't aligned the char* to align with a uint32_t* this code wont work when creating it.
# References
##### Main Notes
[[Compiling shaders with vulkan]]
#### Source Notes
[[Drawing a triangle]]