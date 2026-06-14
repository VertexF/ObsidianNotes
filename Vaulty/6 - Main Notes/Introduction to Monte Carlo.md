2026-06-13 11:28
Status: #baby 
Tags: [[graphics theory]] [[ray tracing]]
# Introduction to Monte Carlo

When solving rendering equation like the **light transport equation** we need a method to go about solving these intergals that can be higher dimensional and discontinuous. Other method to solve intergals break down in this case but the **Monte Carlo** approach scales nicely with any type of intergal that we need to solving when rendering.

What this means is that we just need to solve $f(x)$ for a given intergrand with Monte Carlo with a random value. This gives us an estimated value on a given integral.

When using randomness in algorithms you have two types of categories. **Las Vegas** and **Monte Carlo**. 

**Las Vegas** algorithms are those that use randomness but always give the same return. For example the entry pivot point in Quicksort, the result here being that the array is sorted no matter the value picked. 

**Monte Carlo** algorithms are when you use randomness and you get different values each time, which eventually converge on the correct value on average when you re-run a Monte Carlo more than once on the same input.
# References
##### Main Notes
[[How basic ray tracing works]]
[[Breaking down the light transport equation]]
[[The short comings of a monte carlo approach for ray tracing]]
#### Source Notes
[[Physically Based Rendering - Monte Carlo]]