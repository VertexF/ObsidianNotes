# Reference https://www.computerenhance.com/p/multithreading

This is a very simple concept, if 1 computer can do something at speed, 2 computers can do something at a faster speed. That's how simply it started now CPU have inbuilt 2, 4, 8, 16 or 32 computers in it. This gives CPU the ability to run programs at the same time.

If you can write your code to take advantage of computers in the CPU you get a massive speed up.

Since a computer within a CPU is a core, why isn't this concept called multicore programming? Well a thread is an OS term, and we hand off threads to the OS in hopes it will use more than 1 core. So when you do multithreading programming you need to make sure that the OS understands what you are doing and doesn't add extra headache type stuff.

When you are doing multithreading code, you end up having access to more of the cache. This is because each core has access to the it's own cache, meaning that this can get really fast, since we aren't just getting a multiple computers to share the work we are also getting an increase in cache size for our data to store in.

It's possible that some CPUs are designed around 1 core and use the other cores as a spare computation like thing. Meaning that the memory bandwidth of the cores might be all used up so when you thread the code, it doesn't speed things up by much. When you do thread the code, you do get a little more memory bandwidth BUT it depends on the CPU.

When you push the CPU to fall outside of the memory bandwidth it can get MUCH slower. This is done by increasing the amount of memory needed so it falls out to main memory or something like that. 

The multithreading can get as high as waste if you're working on something with like 96 cores on a server or something. 