2026-04-01 09:19
Status: #baby 
Tags: [[jolt]]
# Jolt with single threaded access

If you're using the single threaded version you can just pointers to the **Body** class and you can use the non-locking variant of the body interface. There are some restrictions however.

1) You cannot read/write to bodies or constraints while **PhysicsSystem::Update** is running. As as soon as the update begins they are all mutex locked.
2) Using the **ContactListener** which are callbacks that happen within the **PhysicsSystem::Update** call from multiple threads. You can only read body data during a callback.
3) Using the **BodyActivationListener** which the same as **ContactListener** you can only ready from it. 
4) You also have **PhysicsStepListener** which are also used from the **PhysicsSystem::Update** from multiple threads. You're responsible when using this that there are no race conditions. In the step listener you can read/write bodies or contraints but you can't add/remove them.
# References
##### Main Notes
[[Jolt with multithreaded access]]
[[Getting started with jolt physics]]
#### Source Notes
[[Architecture of Jolt Physics]]