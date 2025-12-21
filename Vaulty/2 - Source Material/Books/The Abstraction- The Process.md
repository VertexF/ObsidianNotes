# Reference Operating Systems - Three Easy Pieces

##### Intro
The basic technique of sharing the CPU is called **time sharing** which allows other process to use the CPU concurrently, while slowing the CPU down for any particular task. This is called **context switch** which is a the low level **mechanisms**. These are protocols that allow the user to handle multiple processes. 

You also have high level systems that support CPU virtualisation. Like the **scheduling policy** which is part of the **policy** set of algorithms that help the OS make decisions using: 
- **historical information** e.g which program has run more over the last minute?
- **workload knowledge** e.g. what types of programs are run.
- **performance metrics** e.g. is the system optimising for interactive performance, or throughput?
##### The abstraction - a process
A process is just what you call a running program. Think to VS and how you attach a process to debug. You'll want to think of a process the things it's trying to access and is operating on. So we need to worry about the **machine state**
- what can the process read/write to
- what part of the machine are important to the process at a give point in time.

What are the states for the state machine well it's the memory that lives in the address space of the process and the instructions of the process. You have some special registers when it comes to a process.
- **Program counter/instruction pointer** (PC or IP) these tell the program what will execute next.
- **Stack pointer** and **frame pointer** manage the function parameters, local variables and return addresses.
- **I/O information** this is information about files the process has open and stuff.

Typically an OS will have these general API the process will interact with
- **Create** this is for creating the process, typically what happens when you run a program.
- **Destroy** this is for a forceful close of a process.
- **Wait** this is for allowing the process to wait for itself to close.
- **Miscellaneous control** this is for other things that aren't, wait or destroy. Things like suspending the process and resuming it.
- **Status** this get the basic status information on the process running.

The operating system needs to load things in memory before it can run a program. This used to be done **eagerly** where everything is loaded into memory all at once, but now-a-days you have **lazily** which means as the process executed as things are needed, this includes the static data too.

Once the work of the setting up the program has finished, you have the **run-time stack** is allocated all once for the process, this also includes stack variables like argc and argv array. The **heap** is very small and increases as the program requests more. 

The OS will initialise 3 **I/O** file descriptors
- standard input.
- standard output.
- error output.

After loading the code, static data, the stack frame, I/O set up the OS has the set the stage for a program to become a process. The OS finally jumps the **main()** and hands over the CPU to the program to begin execution.