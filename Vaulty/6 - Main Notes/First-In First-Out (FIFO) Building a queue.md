2026-11-11 11:49
Status: #baby 
Tags: [[C++]]
# First-In First-Out (FIFO) Building a queue

Sometimes you want to do the opposite of an array with it's First-In Last-Out (FILO) structure. Do do that you'll need a queue. What's cool about a queue though is that you can build it from an array if you loop through it backward.

Here we are adding things to be an array to be processed later.
```c++
void ParticleBullet::handleEvents()
{
    if (inputHandler.isKeyDown(Keys::KEY_Z))
    {
	    uint32_t index = bulletEntityType[BulletType::BULLET_PLAYER].index;
	    //Now the newest element has highest lifetime value.
	    bulletPool[index].lifetime = 150.f;
	    
        //We are appending bullets to the queue to be worked through in the update loop.
        count.index = index;
        count.type = bulletEntityType[BulletType::BULLET_PLAYER].type;
        bulletQueue.push(count);
    }
}
```

Here we are looping through an array backwards and only popping things off the queue when `.lifetime <= 0.f` 

```c++
void ParticleBullet::update(float delta)
{
    if (bulletQueue.size > 0)
    {
        for (int32_t i = bulletQueue.size - 1; i >= 0; --i)
        {
            uint32_t index = bulletQueue[i].index;

			//Condition from removing the first thing in the queue.
			//Other conditions can be here to remove elements from the queue.
            if (bulletPool[index].lifetime <= 0.f)
            {
                mat4s resetPos = glms_mat4_identity();
                resetPos.m30 = FLT_MAX;
                resetPos.m31 = FLT_MAX;
                resetPos.m32 = FLT_MAX;
                entityData[index].position = resetPos;

                //swap element
                bulletQueue.deleteSwap(i);

				//We have to continue here to avoid updating the velocity, on something that's been removed.
                continue;
            }

			//Here we are doing the maths to update the velocity.
            JPH::BodyID bodyID = bulletPool[index].entity.bodyID;

			//Get velocity
			JPH::Vec3 currentVelocity = bodyInterface->GetLinearVelocity(bodyID);
			JPH::Vec3 desiredVelocity = bulletPool[index].bulletVelocity * 12200.f;
			JPH::Vec3 newVelocity = 0.75f * currentVelocity + 0.25f * desiredVelocity * delta;

			bodyInterface->SetLinearVelocity(bodyID, newVelocity);
			
			entityData[index].position = modelPosition;
			//Now we reduce the lifetime of the object delta time to count down when it should be removed.
			bulletPool[index].lifetime -= fixedTimeStep * delta;
        }
    }
}
```

# References
##### Main Notes
#### Source Notes
