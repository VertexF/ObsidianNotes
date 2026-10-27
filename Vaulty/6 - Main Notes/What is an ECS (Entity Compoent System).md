2026-10-25 09:41
Status: #baby 
Tags: [[Entity system]]
# What is an ECS (Entity Compoent System)

In an ECS a system is something that can declare it's read and write dependencies to get some automatic parallelism. A component system is just something that can add data to an entity at run time.

Has-a is what composition is about in the OOP space it's really not important why it's called that. All it really means on a practical level is that you have classes or structs inside classes and structs

```c++
struct A{};
struct B
{
	A apple;
};
```

Apparent benefits
1) It allows for composition of functionality create emergent behaviour.
2) It's for designers because of the tooling it provides.
3) Has HUGE performant benefits.

These are promoted because of the commerical game engine space but the benefits are more of a half truth.

What's actually true when it comes to game development
1) Not all the benefits are exclusive to ECS's
2) You don't need all the features to ship a game.
3) ECS is a compression algorithm.

Compositions is a properties of an ECS not it's outcome, so if you want composition you don't need an ECS for just that.
# References
##### Main Notes
[[The all in one struct for entities]]
#### Source Notes
[[Entity component system bad]]