# Reference https://www.youtube.com/watch?v=EJP9sXFrKzE

When building Jolt there are 2 version you should be concerned with **release** and **distribution**. Release is the physics engine with profiling and debugging support, distribution is the same but with no debugging or profiling. The reset of the version should be ignored and only for looking for bugs.

For our proposes you will want to got to the build location of **JoltPhysics/Build** in there for our purposes run the bat file **cmake_vs2026_cl.bat** which will generate a build folder.

Assuming you can build and run, in the VS project you should be able to run **Samples** to see what the physics engine can do. The documentation for this engine is https://jrouwe.github.io/JoltPhysicsDocs/
##### Jolt supports
Jolt supports non-deformable shapes such as
1) Spheres
2) Boxes
3) Cylinders
4) Capsules
5) compound shapes - which are a combination of the other shapes
6) Triangle meshes - Used for terrain
7) Height maps - Used for terrain
8) Infinite planes
9) Convex hulls

