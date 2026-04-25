# Reference https://jrouwe.github.io/JoltPhysics/

##### Getting started
There a hello world [HelloWorld](https://github.com/jrouwe/JoltPhysicsHelloWorld) example if you just want to get stuck in. It also has a CMake intergration but we wont likely need that.

If need to educate yourself on how to implement a feature you should check out the samples in the git repo. If you download it from the Internet and run **cmake_vs2026_cl.bat** you should get everything working in a runnable state. Each sample should teach you how to implement a physics idea simulation like basic collision detect or water etc.

When building Jolt there are 2 version you should be concerned with **release** and **distribution**. Release is the physics engine with profiling and debugging support, distribution is the same but with no debugging or profiling. The reset of the version should be ignored and only for looking for bugs.

Assuming you can build and run, in the VS project you should be able to run **Samples** to see what the physics engine can do. The documentation for this engine is https://jrouwe.github.io/JoltPhysicsDocs/
##### Bodies
Jolt uses the traditonal physics simulator bodies which are part of a **Body** class. For collision detection you attached collision shapes part of the **Shape** class.

Bodies can be either
1) **static** (not moving or simulating)
2) **dynamic** (moved by forces)
3) **kinematic** (move by velocities only)

Moving bodies keep track of their state with the **MotionProperties** class object. Static bodies do **NOT** have this this **UNLESS** you set the
`BodyCreationSettings:mAllowDynamicOrKinematic`.
##### Creating Bodies
You can't create **Body** objects yourself. To create a body you need through a **BodyInterfance** class which you create as part of the **PhysicsSystem** class. I believe this is because of threading reasons.

This general life cycle of a body:
1) `BodyInterface::CreateBody` - Creation a body object and initialises it.
2) `BodyInterface::AddBody` - This adds the body to the **PhysicsSystem** and it will now take part in the simulation!
3) `BodyInterface::RemoveBody` - Removes the body from the **PhysicsSystem**.
4) `BodyInterface::DestroyBody` - Does the same thing as remove but deletes the body as well.

If you want to batch create add bodies to the physics simulation you'll want to run these functions.
1) `BodyInterface::AddBodiesPrepare` This doesn't add bodies the simulation but preps the **PhysicsSystem** for all your bodies. Can/should be done in a background thread.
2) `BodyInterface::AddBodiesFinalize` Inserts all the bodies into the simulation, atomically.
3) `BodyInterface::AddBodiesAbort` This is used if you called `AddBodiesPrepare` and now you don't want to add these objects to the physics simulation. Useful to call if the player changes their mind and goes the other way.
4) `BodyInterface::RemoveBodies` Removes batches of bodies from the **PhysicsSystem**

You should alway try to batch bodies creation/adding to the **PhysicsSystem** outside of DoD principles, it can break the broad phase part of the collision detection because the it will run out of internal nodes. If you need to add a lot of nodes it's a good idea to run `PhysicsSystem::OptimizeBroadPhase` to rebuild it's internal tree structure.

You can call AddBody, RemoveBody, AddBody, RemoveBody to temporarily remove and later reinsert a body into the simulation.
##### Multithreaded Access
Just like Vulkan, Jolt is designed to be access from mulitple threads so the **BodyInterface** comes in two flavours a locking and non-locking variants. If you're using the multithreaded version you have to use the multithreading version. The locking variant uses a mutex array. This array is fixed size and has a bunch of mutexes in them and through a hashing process a body instance coorisponds to a mutex, this prevents concurrent access to the same body.

When using the locked version of the called **BodyLockInterface** you refer to bodies with a **BodyID**. You would access a body via this psudecode.

```c++
JPH::BodyLockInterface lockInterface = physicsSystem.GetBodyLockInface();
JPH::BodyID bodyId = ...; //Obtain the ID from the body.

//Here we are creating a scope lock.
{
	JPH::BodyLockRead lock(lockInterface, bodyId);
	if(lock.Succeeded()) //bodyId may no longer be valid.
	{
		const JPH::Body& body = lock.GetBody();
		//Do something with the body 
	}
}
```

If another thread removes the body the by the time you're between getting the `JPH::BodyID` and `JPH::BodyLockRead lock(lockInterface, bodyId);` the lock will fail. Each BodyID contains a sequence number, so body ID's will be reused after many add/remove cycles. To write to a body you need to use the **BodyLockWrite** class.

If you need to read/write a group of Bodies you need to use the **BodyLockMultiRead** or **BodyLockMultiWrite**. The reason is that if two threads lock the bodies in the opposite order you set them to be locked, you'll get a dead lock.

You'll have a lot of convenience functions are exposed through the **BodyInterface**, but not all functionality is available but you'll need to lock the bodies to access them and use the functions they have to use everything.
##### Single threaded access
If you're using the single threaded version you can just pointers to the **Body** class and you can use the non-locking variant of the body interface. There are some restrictions however.

1) You cannot read/write to bodies or constraints while **PhysicsSystem::Update** is running. As as soon as the update begins they are all mutex locked.
2) Using the **ContactListener** which are callbacks that happen within the **PhysicsSystem::Update** call from multiple threads. You can only read body data during a callback.
3) Using the **BodyActivationListener** which the same as **ContactListener** you can only ready from it. 
4) You also have **PhysicsStepListener** which are also used from the **PhysicsSystem::Update** from multiple threads. You're responsible when using this that there are no race conditions. In the step listner you can read/write bodies or contraints but you can't add/remove them.

If you're multithreaded, you should use **BodyID's** instead of bodies. You can use the **Body** class, you just have to be extremely careful. You have to clear the **Body** at the right time. Also you need to make sure you do no read/write during the **PhysicsSystem::Update** while other threads are reading the **Body**, basically you're in charge of avoiding race condition.
##### Shapes
Each **Body** has a shape attached that determines the collision volume, here we have the shapes in order of computational complexity.

1) **SphereShape** - A sphere centred around zero.
2) **BoxShape** - A box centred around zero.
3) **CapsuleShape** - A capsule shape centred around.
4) **TaperedCapsuleShape** - A capsule shaped with a different radii at the bottom and top.
5) **CylinderShape** - A cylinder shape. Note that the cylinders are the least stabel of all the shapes, so use another if possible.
6) **TaperedCylinderShape** - A cylinder with different a different radii at the top and bottom. Also unstable like **CylinderShape**.
7) **CovexHullShape** - A covex hull defined by a set of points.
8) **TriangleShape** A single triangle. Use **MeshShape** if you have multiple triangles.
9) **PlaneShape** - An infinite plane negative hald space is considered solid.
10) **StaticCompoundShape** - As shape containing other shapes. This shape is constructured once and cannot be changed afterwards. Child shapes are organised in a tree to speed up collision detection. 
11) **MutableCompoundShape** - Like **StaticCompoundShape** but can be changed at runtime. The shapes are in a list so you can do this easier. Note it is slower.
12) **MeshShape** - A shape consisting of triangles. They are mostly used for static geometry.
13) **HeighFieldShape** - A shape consisting of NxN point that define the height at each point. very suitable for representing hilly terrain. Any body that uses this shaope needs to be static.
14) **EmptyShape** - As that collides with nothing that be used as a placeholder.

Next to this you have a number decorator shape that changes the behavior of their children.

1) **ScaledShape** - This shaope can scale a child shape. Be careful, do not rotate **THEN** scale you might sheer a shape which isn't supported in jolt.
2) **RotatedTranslatedShape** - This shaope can rotate and translate a child shape. Might be used if you need to move something like a **SphereShape** from the origin.
3) **OffsetCenterOfMassShape** - This doesn't change the shape of the child but changes were the centre of mass is calculated. Helps with vehicles to help with handling.
##### Dynamic Mesh Shapes
