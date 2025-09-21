2025-09-21 07:56
Status: #adult 
Tags: [[vulkan]] [[vulkan validation]]
# What are validation layers

Vulkan is designed to have very little driver overhead. Meaning that is is next to no error checking when using vulkan without layers. 

The validation layer sits between vulkan function calls and driver and does error checking over these main categories:

- Checking values for arguments and tracing, profiling and replying functions calls. 
- Keeping track of the creation of that thread and what it's up to for thread safety.
- It check the creation and deletion order of objects is create, avoid memory leaks.
- Logging all the errors and none errors to standard out or error out streams.

Laying validation like this is handy because they can be completely removed when you release your software, and validation **SHOULD** always be turned off in release.

LunarG SDK for vulkan has amazing layer validation you can use. Be careful it's not perfect and mistakes can and will slip through validation so keep updating your SDK when possible.
# References
##### Main Notes
[[What is Vulkan]]
[[Vulkan Layers aren't extensions]]
[[Instance and device level validation]]
[[Enabling Validation layers]]
#### Source Notes
[[Vulkan-Tutorial]]