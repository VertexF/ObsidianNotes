
https://www.youtube.com/watch?v=ZwM2kMKgtdU - It all just practice

# Learning C++ for game engine development
You want to learn C++ really well otherwise you wont be able to do much when you try to apply it to graphics. You don't have to learn everything but I should be able to quiz you on the first barrier to entry and you should be able to answer in some detail.

I recommend taking up a small project from https://www.youtube.com/@javidx9 while learning C++ to keep yourself motivated they are AMAZING for learning, fun and short game related projects. Pick something that interests you and learn about it. Pick a C++ books likely the best way to learn C++ anything will be good enough if it teaches C++ from the beginning, learncpp.com is a great website if you don't like learning books.

### C++ links and books
learncpp.com - Go through this site from top to bottom. The tutorial assumes nothing. You'll learn everything to get started with this.
https://stroustrup.com/tour3.html - Long and complete overview for learning C++
https://stevedewhurst.com/cpp_gotchas/index.html - C++ Gotchas - Great to learn some of the common pitfalls 
https://www.youtube.com/watch?v=18c3MTX0PK0&list=PLlrATfBNZ98dudnM48yfGUldqGD0S4FFb - Good series to help with topics you don't understand in C++
Effective C++: 55 Specific Ways to Improve Your Programs and Designs - Good for getting to gribs with how to use C++ better.

# Computer fundimentals
Learn at least some foundational computer science topics on memory.
https://www.geeksforgeeks.org/c/memory-layout-of-c-program/
https://www.geeksforgeeks.org/dsa/stack-vs-heap-memory-allocation/
https://www.geeksforgeeks.org/operating-systems/memory-management-in-operating-system/
https://pages.cs.wisc.edu/~remzi/OSTEP/ - Operating System : Three Easy Pieces - Books: Part 1 and 2 are great for this pretty easy to follow, requires Linux of WLS for Windows users.

### Barrier to entry -
Before moving on to something if you're using C++ know all this for 3D game engine development 
1) Basic types (int, floats, doubles) and how big they are.
2) All control structures (if, switch)
3) All the loop types and when they are used.
4) Data structures. This would include learning the standard C++ STL data structures like std::vector, std::stack and std::map etc.
5) Dynamic memory (what is the stack and heap is) how to allocate dynamic memory, why is this used, what is it used for, when do you want to use it.
6) Pointer, reference, double pointers, pointer to reference.
7) All casting types (static_cast, c-style cast, dynamic_cast etc) 
8) Arrays, (static and dynamic) maps (ordered and unordered), queues and stacks are going to be the most important data structures, to know and understand well. 
9) Functions, Classes and structs basics. 
10) Arguments, pass by value, pass by reference, pass by pointer. Pros and cons of these.
11) pre-process things like marcos and defines.
12) Learn all 3 different version of uses of static and how the global state works when runnings a program.
13) learn about inline
14) Learn the Singletons design pattern.
15) Inheritence, polymorphism, composition and the OOP is/has relationships you can build up between different objects.
16) Learn how to debug your program with break points. Learn how to use 1 debugger.
17) learn what are common runtime issues you can come into are like deferencing nullptrs, read access violation, write access violation.
18) Basic constness, like const functions, const pointers, const references, const values, constexpr.
19) Basic expections handling.
20) asserts static and runtime asserts. 
21) Learn the different C++ compilers.
22) Basic templates, nothing crazy, you should be able to write and understand basic function based and class based templates.
23) Know how to use lambdas
24) Learn what libraries are, what is the difference between dynamic and static libraries for example.
25) Learn what is sizeof() how big structure and classes are 
26) Learn what Operator overloading is and how to write your own.
27) Learn how the constructor and destructor works
28) All scope types.
29) File I/O.
30) Have build a medium size project before.
31) A good understand of how to your IDE.
32) Learn how computes work at a good deep level.

Note this isn't a complete list. It's just the bare minimum you will need, I should be able to give you on these in some detail.

# Introduction to how to begin with 3D game development.
Now for 3D game engine development I would first focus on a renderer. A renderer takes roughly 60 to 70% of the frame to do something with. This means that most of your optimisation and performance work will be going into graphics over everything else. With that in mind you have 2 options.

Vulkan first or OpenGL first then Vulkan afterwards (if you want to learn Vulkan). They both have pros and cons. 

# Vulkan Path pros and cons
Cons - If you learn Vulkan first you're going to have a harder time getting things rendering Please consider if you might not even need Vulkan for your 3D renderer anyway if you're code is simply enough. The learning curve is much steeper because you have to worry about hardware synchronisation on top of learning graphics.

Pros - OpenGL is VERY difficult to get working in a multithreaded environment. Graphics drivers have to do less work to with Vulkan because you're coding all that yourself. Meaning that your code will be likely be more consistent if you write your vulkan code correctly. Vulkan is getting new features and updates and will like be one of the main industry standards for many years to come, OpenGL will still work for decades but is getting old.

### Vulkan Links
Vulkan resource - Make sure all your vulkan resources are working with Vulkan 1.3
AVOID DO NOT USE THIS -> https://vulkan-tutorial.com/ 
Vulkan Discord server https://discord.gg/vulkan
https://www.howtovulkan.com/ - This is good introduction
https://vkguide.dev/ - Another introduction just more in depth
Vulkan 3D Graphics Rendering Cookbook - A very gental introduction with something cool recieps 
Mastering graphics with Vulkan - Pre vulkan 1.3 but has great techniques
Some older and newer Vulkan tutorials https://www.youtube.com/@OGLDEV
https://www.youtube.com/@zeuxcg for more advanced mesh shading stuff, use this over other things listed for mesh shading.

# OpenGL pros and cons 

Cons - You're not learning Vulkan. Not all the knowledge of OpenGL transfers to Vulkan and you'll still have a steep learning curve to overcome with Vulkan. You can't easily multithread OpenGL. It's older, so it doesn't have all the brand new graphics features like Vulkan. 

Pros - Easier to learn and your going to have an easier to time getting some nice graphics on the screen. 

### OpenGL links
https://learnopengl.com/ - The standard for learning OpenGL you'll learn a lot of nice graphics theory and basics very quickly. The maths isn't perfect but it's good enough for gettting started.
https://www.youtube.com/@OGLDEV - More videos on OpenGL that have pretty nice techniques.

### Barrier to entry -
You need to be able to render a basic models with different textures and materials.
You also need a game noclip style camera so you can fly around your model to see it clearly.

If you want to make nice graphics appear on your screen you're going to have to learn the theory. This is mostly shader programming and is even relevant for game engines like Unreal and Unity. Build this up as you go please don't feel like you're looked b

# Maths for graphics theory
https://www.youtube.com/watch?v=DPfxjQ6sqrc - A super basic introduction to game dev maths.
https://www.youtube.com/watch?v=fNk_zzaMoSs&list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab - Linear algebra is the foundation of EVERYTHING this playlist is worth going through carefully to learn Linear Algebra.
Foundations of Game Engine Development Volume 1 - A great maths book to get you started with serious game development maths.
https://www.youtube.com/watch?v=2hBWCCAiCzQ&list=PLVuwZXwFua-0Ks3rRS4tIkswgUmDLqqRy - More geometric algebra content that should build off the Foundations of Game Engine Development Volume 1 book

# Graphics theory
https://discord.gg/graphicsprogramming - Graphics programming discord server.
Foundations of Game Engine Development Volume 2 or Real time rendering - Both are pretty complete resources on the foundational graphic techniques
https://google.github.io/filament/main/filament.html - At least do everything for the standard BRDF physical based rendering and learn point light, directional light and spot light if you haven't already.
https://www.pbrt.org/ - If you want to build an offline renderer that's purely a ray tracer. Not really game engine stuff.
GPU ZEN 03 and 04 - Advanced Rendering Techniques - These are good for more advanced folk who want to break into more advanced techniques.

Nice supplements to help with graphics theory
https://www.youtube.com/@simondev758 - Great videos Simon goes over a lot of great techniques.
https://www.youtube.com/@Acerola_t - Great videos on more graphics techinques, advice and such.
https://www.youtube.com/@oskar_schramm - More great graphics techinques videos about things.

### Barrier to entry -
Render more than 1 model
Render some lighting in your scene

# Making the renderer into a game engine.

Now you're going to have to make the rest of the game engine. I would start with physics so you can make the 3D game engine playable this will be a large part of the games frame too like another 10~ish percentage.

# Physics
I personally have made physics engine in the pass and it took me 4 years to do everything from 3D rotation and translation to resolving the collisions. I wouldn't recommend writing your own unless you want to drive into that rabbit hole for a long time.

Here are the 3 Physics engine everyone uses
https://github.com/NVIDIAGameWorks/PhysX - Toykospliff uses this in his Hell engine. I don't know much about it or how well it works.
https://github.com/jrouwe/JoltPhysics - I'm using Jolt Physics it's good if not a little OOP for my liking.
https://github.com/erincatto/box3d - Brand new physics engine. Everyone liked Box2D this will likely be great too, expect bugs though because of how new it is.

# Game engine resource
https://www.gameenginebook.com/ - Take this seriously you'll thank yourself in the future, this IS the road map for making game engines. This is the TRUE roadmap to learning game engine development.
https://www.youtube.com/watch?v=WwkuAqObplU and https://www.youtube.com/watch?v=rX0ItVEVjHc are fantastic for learning about data oriented design which is a programming style that helps you write cache friendly code. It's different that OOP and much better in my opinion for game engine development.

# Other ways to learn Game engine development 
https://www.youtube.com/@MollyRocket - Casey has a 662 playlist on making a game engine from scratch and other great programming content
https://www.youtube.com/@TravisVroman - Travis has a great series were he does educational livestream on making a game engine from scratch in vulkan and C

Of course there is more to learn but you're likely going to be spending thousands of hours on this so it's going to take time. So take your time and learning things well. Remember I can't do the learning for you and I'm still on the journey so don't give up and keep at it.