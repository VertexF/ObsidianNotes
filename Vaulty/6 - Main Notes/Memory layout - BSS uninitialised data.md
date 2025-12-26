2025-12-25 07:33
Status: #baby 
Tags: [[OS]] [[OS memory]]
# Memory layout - BSS uninitialised data

![[memorylayout.png]]

This piece of memory is static at run time. Variables are stored in here when you have is static variables and you have **NOT** initialised them to something. The **block starting symbol** BSS space initalises the static variables at run time with an ASM instruction. 

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
# References
##### Main Notes
[[Memory layout - Text section]]
[[Memory layout - Initialised data]]
[[Memory layout - The heap]]
[[Memory layout - The stack]]
#### Source Notes
[[Memory Layout of a C Program]]
