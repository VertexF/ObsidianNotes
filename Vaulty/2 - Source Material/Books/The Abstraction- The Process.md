# Reference Operating Systems - Three Easy Pieces

##### Intro
The basic technique of sharing the CPU is called **time sharing** which allows other process to use the CPU concurrently, while slowing the CPU down for any particular task. This is called **context switch** which is a the low level **mechanisms**. These are protocols that allow the user to handle muiltple processes. 

You also have high level systems that support CPU virtualisation. Like the**scheduling policy** which is part of the **policy** set of algorithms that help the OS make decisions using: 
- **historical information** e.g which program has run more over the last minute?
- **workload knowledge** e.g. what types of programs are run.
- **performance metrics** e.g. is the system optimising for interactive performance, or throughput?
##### The abstraction - a process
A process is just what you call a running program. Think to VS and how you attach a process to debug. You'll want to think of a process the things it's trying to access and is operating on. So we need to worry about the **machine state**
- what can the process read/write to
- what part of the machine are important to the process at a give point in time.

