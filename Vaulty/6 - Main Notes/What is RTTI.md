2026-04-09 07:23
Status: #baby 
Tags: [[C++]]
# What is RTTI

RTTI - means run time type information. This is needed to be able to tell what the object is at run time. This is used when you're working with polymorphism/inheritance.

It's used when you have a parent and a base with virtual functions. The RTTI allows C++ to work out the object type based on if they are compatible based on the v-table.

You have two different concepts linked to this **Upcasting** and **Downcasting**.

1) **Upcasting** means you are converting a child class to a parent class. This is 100% safe to do because the child class has everything a parent class has.
2) **Downcasting** means you are converting a parent class to a child class. This requires us to do a runtime check to make sure these are valid.
# References
##### Main Notes
[[How dymanic_cast actually works]]
#### Source Notes
[[RTTI (Run-Time Type Information) in C++]]