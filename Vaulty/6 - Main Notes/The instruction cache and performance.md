2026-03-19 17:10
Status: #baby 
Tags: [[CPU]] [[performance]]
# The instruction cache and performance

People don't really talk about working with the instruction cache because it's very rarely the bottleneck. You can do this, by making your loops short to not evict things from your i-cache and also try to use less instructions. If you unroll a loop too much for example you can evict the i-cache of instructions making things slower. Most of the time you'll have bigger problems before that point.

Modern CPUs have a lot more decoders running in parallel to decode instruction that then get stored in a trace cache if necessary. Each decoder can work on different part of the instruction too, so they aren't always working on separate instructions. 
# References
##### Main Notes
[[Introduction to the instruction cache]]
#### Source Notes
[[Casey Q&A 4]]