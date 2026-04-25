2026-04-01 09:07
Status: #baby 
Tags: [[jolt]]
# Basics of jolt bodies

Jolt uses the traditonal physics simulator bodies which are part of a **Body** class. For collision detection you attached collision shapes part of the **Shape** class.

Bodies can be either
1) **static** (not moving or simulating)
2) **dynamic** (moved by forces)
3) **kinematic** (move by velocities only)

Moving bodies keep track of their state with the **MotionProperties** class object. Static bodies do **NOT** have this **UNLESS** you set the `BodyCreationSettings:mAllowDynamicOrKinematic`.
# References
##### Main Notes
[[Creation of jolt bodies]]
[[Getting started with jolt physics]]
#### Source Notes
[[Architecture of Jolt Physics]]