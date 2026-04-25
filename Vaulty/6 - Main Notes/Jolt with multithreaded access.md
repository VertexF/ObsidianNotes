2026-04-01 09:14
Status: #baby 
Tags: [[jolt]]
# Jolt with multithreaded access

Jolt is designed to be access from mulitple threads so the **BodyInterface** comes in two flavours a locking and non-locking variants. If you're using the multithreaded version you have to use the locking version. The locking variant uses a mutex array. This array is fixed size and has a bunch of mutexes in them and through a hashing process a body instance corresponds to a mutex, this prevents concurrent access to the same body.

When using the locked version of the called **BodyLockInterface** you refer to bodies with a **BodyID**. You would access a body via this pseudocode.

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

You should use **BodyID's** instead of bodies. You can use the **Body** class, you just have to be extremely careful. You have to clear the **Body** at the right time. Also you need to make sure you do no read/write during the **PhysicsSystem::Update** while other threads are reading the **Body**, basically you're in charge of avoiding race condition.
# References
##### Main Notes
[[Jolt with single threaded access]]
[[Getting started with jolt physics]]
#### Source Notes
[[Architecture of Jolt Physics]]