# Reference www.computerenchance.com/p/welcome-to-performace-aware

This course isn't meant to teach you how to write code that is hyper optimised, but code that isn't very very slow. Hence you'll become aware of performance. So you'll be learning how performance work and this will give you the ability to make correct decisions on how to write code.

Performance is really simple, we are talking about how long the CPU in total takes to complete instructions. There are 2 things we can do as programmers to really speed things.

1) **Reduce** the number of instructions. With the cavate that sometimes this doesn't always make things faster.
2) **Speed** of instructions on the CPU. This isn't talking about changing the clock rate of the CPU. We are talking about how long an instruction takes to complete on the CPU. It's possible that 1 instruction takes 1 cycle complete, other could take literal 100's to complete. 

For speed we are concerned with what the instructions are, how they access memory and the order of the instructions.

Programmers were once deeply aware of the instructions of the CPU had to do, now programmers aren't aware making programs slower. In the 90's CPU got way more complex, so they got harder to understand so programmers pretty much started to not learn the instructions.

We are trying to include the in our thinking while we code is the instructions that the compiler generated for the CPU because that's actually what's going to run on the CPU. 

If you learn this stuff
1) You'll be able to make better decisions
2) You'll know how fast something should be. Meaning that when you write code and it's slow you'll notice that.