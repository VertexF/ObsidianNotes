2025-12-25 07:29
Status: #baby 
Tags: [[OS]] [[OS memory]]
# Memory layout - Initialised data

![[memorylayout.png]]

This piece of memory is static at run time. Variables are stored in here when you have is static variables and you **HAVE** initialised them to something. This can be in the static local scope or static globally scope variables that are static 

```c++
#include <stdio.h>

// Global variables (stored in initialized data segment)
int globalVar = 10;
char message[] = "Hello";

int main()
{
    // Static variable (also stored in initialized data segment)
    static int staticVar = 20;

    printf("Global variable: %d\n", globalVar);
    printf("Static variable: %d\n", staticVar);
    printf("Message: %s\n", message);

    return 0;
}
```
# References
##### Main Notes
[[Memory layout - Text section]]
[[Memory layout - BSS uninitialised data]]
[[Memory layout - The heap]]
[[Memory layout - The stack]]
#### Source Notes
[[Memory Layout of a C Program]]