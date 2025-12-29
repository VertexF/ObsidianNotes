2025-12-29 11:05
Status: #baby 
Tags: [[OS]] [[CPU]]
# High level overview on CPU virtualisation

The basic technique of sharing the CPU is called **time sharing** which allows other [[What is a process|process]] to use the CPU concurrently, while slowing the CPU down for any particular task. This is called **context switch** which is a the low level **mechanisms**. These are protocols that allow the user to handle multiple processes. 

You also have high level systems that support CPU virtualisation. Like the **scheduling policy** which is part of the **policy** set of algorithms that help the OS make decisions using: 
- **historical information** e.g which program has run more over the last minute?
- **workload knowledge** e.g. what types of programs are run.
- **performance metrics** e.g. is the system optimising for interactive performance, or throughput?
# References
##### Main Notes
[[What is a process]]
#### Source Notes
[[The Abstraction - The Process]]
