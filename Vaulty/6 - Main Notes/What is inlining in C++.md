2026-07-10 09:11
Status: #baby 
Tags: [[C++]]
# What is inlining in C++

Inline outside of a class means that you can violate [[What is the one definition rule|ODR(One definition rule)]] Meaning that we have multiple definitions across different translations units. With making it static it has external linkage.

This allows us to have static storage duraction. If an inline variable is in the scope of a class then it behaves like a static member variable and we can initialise this within class. We can in C++17 have different definitions of inline variable in translation units creating new copies of these varaible.

Static means that you have globals that are instantiated for every translation unit. The thing goes for function as well. Statics might get duplicated in the binrary (depending on what LLVM decided to do.)
# References
##### Main Notes
[[What is the one definition rule]]
#### Source Notes
