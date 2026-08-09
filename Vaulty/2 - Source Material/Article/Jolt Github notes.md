# Reference https://github.com/jrouwe/JoltPhysics/discussions/1475

##### Jolt Body and reference counting
In Jolt the `Body` class is not reference counted because it requires very careful memory management. You can access this fella from any thread. You can't just call delete on `Body` because internally `(SoftBody)MotionProperties` are allocated in the same memory block, all for performance.

You can override Jolts `new`/`delete` can be overridden in Jolt but you need to be very careful here for what we described above. You need to be able to access bodies in any thread.

When you have a reference counted object it can be shared between unrealted objects.

```c++
Ref mesh = new MeshShapeSettings(....);

Ref compound1 = new StaticCompoudShapeSettings;
compound1->AddShape(<some orientation>, mesh);
compound1->AddShape(<other orientation>, mesh);

BodyCreationSettings body1(compound1, ...)
<create the body>

Ref compound2 = new StaticCompoudShapeSettings;
compound2->AddShape(<an orientation>, mesh);

BodyCreationSettings body2(compound2, ...)
<create the body>
```

As you can see here we are sharing `Ref mesh` around. This is nothing out of the ordinary for reference counted objects. This allows for reuse in Jolt. This is used everywhere in Jolt.

You need to make your allocator thread safe because of careful things are allocated within Jolt.