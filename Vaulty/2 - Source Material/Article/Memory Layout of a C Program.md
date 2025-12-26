# Reference https://www.geeksforgeeks.org/c/memory-layout-of-c-program/

I will go over each part of this picture.

![[memorylayout.png]]

##### Text/Code section
This is static at run time. This is were the instructions and string literals live within your program when they need to be ran. This is fixed in size and is read-only to prevent accidently overwriting. 
##### Data section (Initialised/Uninitialised section)
This is the initialised and non-initialised, part of memory in a program. These are static at run time. 
##### Initialised data
Is data that is static and initialised to something. This can be in the local scope or globally scope variables that are static. 

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
##### Uninitialised data
This piece of memory is for static variables that aren't initialised. The **block starting symbol** BSS space initalises the static variables at run time with an ASM instruction. 

These are BBS segment variables.
```c++
#include <stdio.h>

// Global uninitialized variables (stored in BSS segment)
int globalVar;
char message[50];

int main()
{
    // Static uninitialized variable (also stored in BSS)
    static int staticVar;

    // Assigning values at runtime
    globalVar = 10;
    staticVar = 20;
    snprintf(message, sizeof(message), "Hello BSS");

    printf("Global variable: %d\n", globalVar);
    printf("Static variable: %d\n", staticVar);
    printf("Message: %s\n", message);

    return 0;
}
```

You have to be careful because the initialisation is guaranteed to happen before any stack variable so it's safe to use there BUT the order in which these are initailsed is somewhat random per translation unit. So if one piece of static data relies on another piece of static to be initialised and they are both in the **block starting symbol** then a crash can happen.

```c++
1. // File x.cpp
2. #include "Fred.h"
3. Fred x;

//The file `y.cpp` defines the `y` object:

4. // File y.cpp
5. #include "Barney.h"
6. Barney y;

//For completeness the `Barney` constructor might look something like this:

7. // File Barney.cpp
8. #include "Barney.h"

9.  Barney::Barney()
10. {
11.   // Here we are assuming that the Fred class is initialised 
12.   // which isn't always true. This will crash sometimes.
13.   x.goBowling();
14.   // ...
15. }
```

If you need a static variable that depend on other use the **Construct On First Use Idiom** aka **Singleton** pattern. 

```c++
1. #include "Fred.h"

2. Fred& x()
3. {
4.   static Fred* ans = new Fred();
5.   return *ans;
6. }

7. // File Barney.cpp
8. #include "Barney.h"

9. Barney::Barney()
10. {
11.   // ...
12.   x().goBowling();
13.   // ...
14. }
```

Why not use initialised data all the time instead of this run-time initialised thing that has issues? Well image you have an array huge like `static int particles[120000]` and I set them to zero with a `static int particles[120000]{0};` everything in this array would have to be stored directly in the **initialised** part of the executable. Since this a simple array but big we can just rely on the ASM instruction to make them all 0 at time saving space.
###### The heap
This is dynamic piece of memory that starts after the BSS and grows larger with every request to allocate memory at run time. This shinks as heap memory is dellocated.
##### The stack
This is a piece of dynamic memory that grows as more things are added to the stack. These are variables like local variables, function parameters, and return addresses. Each function add a stack frame when the function is ran, it then pops off the stack once the function has finished. This is done by a stack pointer that moves up as things are added and moves down as things are popped. Which is why it's called the stack it's FIFO.

When the stack and the heap meet the program is out of free memory.