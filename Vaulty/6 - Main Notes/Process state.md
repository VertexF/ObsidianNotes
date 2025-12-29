2025-12-29 11:12
Status: #baby 
Tags: [[OS]] [[CPU]]
# Process state

There are 5 overview states a process:
- **Initial** This is when the OS is starting up the process.
- **Running** This means the process is running instructions
- **Ready** This means the process is ready to go but the OS has stopped the process for a moment.
- **Blocked** This means that the process has requested something that requires an event to be taken place. This can be something like request some I/O interaction with something like a file.
- **Final/Zombie** this is when a process has finished and has yet to be cleaned up.

![[process-states.png|439x349]]

As you can see in this picture this how a process can transition from one state to another.

Typically when a process is stopped because the schedular part of the operating system will run another process. 

The **final** state is useful because you can have a parent process that is waiting for the **final** state to read the turn code of the child process, to then call a `wait()` so the OS can clean up.
# References
##### Main Notes
[[What is a process]]
#### Source Notes
[[The Abstraction - The Process]]