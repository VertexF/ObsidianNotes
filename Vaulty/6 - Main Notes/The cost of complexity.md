2026-10-25 09:51
Status: #baby 
Tags: [[Entity system]]
# The cost of complexity

This is often completely over looked but there is a hidden cost to increasing the complexity of software. If you're an indie you really need to be careful to solve the problems you actually have, you simply do not have the time to solve a therically problem that by turn up, like too much RAM usage for example, is this actually a problem for you? 

So if I have a [[The all in one struct for entities|all in one struct]] and you're concerned that a couple of extra data per entity data field that causes memory wastage. Please measure if you're actually going to run out of memory because you have couple of wasted variables.

Why is this important? You want to start off solving the most common case first not the most generic problem. Meaning you don't want to generalise or optimise too early. I've personally made both mistakes and it always comes back to haunt you with increased complexity.

These compressed data things cost you by increasing the complexity of your game. It's extremely important to note that you don't have as much time as yout think so you should only go out to solve problems you do have or you'll **DEFINITELY** will add more problems you'll have to solve.
# References
##### Main Notes
#### Source Notes
[[Entity component system bad]]