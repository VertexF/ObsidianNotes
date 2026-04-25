2026-04-01 09:09
Status: #baby 
Tags: [[jolt]]
# Creation of jolt bodies

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
# References
##### Main Notes
[[Basics of jolt bodies]]
#### Source Notes
[[Architecture of Jolt Physics]]