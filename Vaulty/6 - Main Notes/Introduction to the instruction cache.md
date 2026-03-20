2026-03-19 17:06
Status: #baby 
Tags: [[CPU]]
# Introduction to the instruction cache

The registers don't store the instruction. Registers are only about data, there are some instruction decode caches which are like registers for instructions in newer CPUs. Instructions flow through a queue, but you also have trace caches.

Trace caches are used when,
1) You decode the instruction
2) You store that micro-operation in a cache if it's likely going to be used a lot.
3) Then we re-fetch from the trace cache.
# References
##### Main Notes
[[The instruction cache and performance]]
#### Source Notes
[[Casey Q&A 4]]