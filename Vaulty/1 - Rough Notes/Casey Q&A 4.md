The registers don't store the instruction. Registers are only about data, there are some instruction decode caches which are like registers for instructions in newer CPUs. Instructions flow through a queue, but you also have trace caches.

Trace caches are used when,
1) You decode the instruction
2) You store that micro-operation in a cache if it's likely going to be used a lot.
3) Then we re-fetch from the trace cache.

There is a memory to memory move on the CPU in the 8086 at least. These do exist but not in the **mov** these exist in the **rep**-prefix instructions and they handle moving memory around on main memory.

People don't really talk about working with the instruction cache because it's very rarely the bottleneck. You can do this, by making your loops short to not evict things from your i-cache and also try to use less instructions. This can actually happen, if you unroll a loop too much. Most of the time you'll have bigger problems before that point.

Up to skylake intel CPUs at least, you can natively run 16-bit instructions. However you can't just run them without using backwards compability thing. With something like hypervisor you can put the CPU in 8086 mode and it will start to execute 8086 instructions. It's a kernel level thing it's protected so you need the OS to put you in that mode.

Modern CPUs have a lot more decoders running in parallel to decode instruction that then get stored in a trace cache if necessary. Each decoder can work on different part of the instruction too, so they aren't always working on separate instructions. 

