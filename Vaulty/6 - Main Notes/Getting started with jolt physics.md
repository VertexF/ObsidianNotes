2026-04-01 08:57
Status: #baby 
Tags: [[jolt]]
# Getting started with jolt physics

There a [HelloWorld](https://github.com/jrouwe/JoltPhysicsHelloWorld) example if you just want to get stuck in. It also has a CMake intergration but we wont likely need that.

If you need to educate yourself on how to implement a feature you should check out the samples in the git repo. If you download it from the Internet and run **cmake_vs2026_cl.bat** you should get everything working in a runnable state. Each sample should teach you how to implement a physics idea simulation like basic collision detect or water etc.

When building Jolt there are 2 version you should be concerned with **release** and **distribution**. Release is the physics engine with profiling and debugging support, distribution is the same but with no debugging or profiling. The reset of the version should be ignored and only for looking for bugs.

The documentation for this engine is https://jrouwe.github.io/JoltPhysicsDocs/
# References
##### Main Notes
[[Jolt with multithreaded access]]
[[Jolt with single threaded access]]
[[Creation of jolt bodies]]
[[Basics of jolt bodies]]
#### Source Notes
[[Architecture of Jolt Physics]]
