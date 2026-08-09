# Reference GPU Zen 3 - Page 123/124

##### Introduction
Particle systems haven't changed since the 2010's as most only render a couple few hundred with the exception of rain, which runs on the GPU. Most particle systems aren't GPU based which reduces performance. 

If you have a GPU based approach you can do something like 1 million particles. So this introduces a compute shader approach that limits the amount of interaction with other shaders to avoid moving things through memory.
##### Suggested high view of particle system.
What you could instead is use instance rendering and update the instance buffer in a compute shader, (assume that we get the buffers correctly sync-ed with memory barriers). Alex (someone from the livestream) collides when with a depth buffer and renders them after the update.

This allows for you to do things like change instance to point different UV and get a nice splash animation on rain, when a collision detection is found. I believe this is because we are using the depth buffer to here just colliding with whatever it hits. You might need to do multi-indirect draw calls to pull.
##### Visibility buffer
This book references a visibility buffer which culls triangles from geometry and if there are no triangles left in for draw call, the draw gets cull too. It does things in 2 phases 1 of translucent materials and opaque materials. It builds a buffer with 1 bit for working out if a triangle is alpha masked, 8 bits for a storing the draw call ID and the last 23 bits store the triangle ID relative to the draw ID.

I believe this is a poors mans mesh shading approach to culling triangles from a geometry, meaning that I don't think I need to do this for a GPU driven particle system.
##### Data Management
You can define a **particle system** as a collection of **particle sets**. A **particle set** is just a set particles that share the same properties likes textures, size, max speed or AI behavior. Each particle within a particle set keeps track of **position**, **age** and **velocity**. Multiple particle sets can achieve more complex effects.

```c
struct Particle
{
	uvec2 velocityAndAge;
	uvec2 position;
};

struct ParticleSet
{
	//x, y, z is the volume of the particle set, w is for scaling particles.
	vec4 size;
	//position of the particle
	vec3 position;
	//Amount of particles to be spawend per second
	float particlesPerSecond;
	//Colour of the particles in the set
	vec3 colour;
	//Radius of emitted light
	float lightRadius;
	//AI type of particles.
	uint particleType;
	//Descripes whether a particle should cast shadows or emit lights
	uint lightBitField;
	//Maximum amount of particles to allocate
	uint maxParticles;
	//Lifetime of the particle.
	float initialAge;
	//Forces for the flocking algorithms.
	uint boidAvoidSeekStrenght;
	uint boidSeparationFleeStrength;
	uint boidCohesionAlignmentStrength;
	uint steeringStrengthMaxSpeed;
};
```

All particles are stored in the same buffer. So we have different buffer that stores a 32-bit field which keep track of whether a things like if particles emits light and what particle set it belongs to. This is an optimisation that allows is to not access the whole particle data. For example if we have a light clustering shader it can detect if a particle is dead by just looking at the bitfield without unpacking the entire particle which is belongs. We need to make sure that our bitfield buffer and our particle are 1 to 1 meaning that if we access bitfield $i$th then that should corrispond to the same index in the particle buffer.

Here is the defines for the bitfield buffer.

```java
//Each bitfield type has a mask that allows us to isolate a
//certain property, in this case the allocation state.
#define BITFIELD_ALLOCATION_BITS_MASK 0x1FFFFU
#define BITFIELD_SET_INDEX_MASK 0xFFFFU
#define BITFIELD_IS_ALLOCATED 0x10000U
//State of the particle
#define BITFIELD_STATE_BITS_MASK 0xE0000U
#define BITFIELD_IS_ALIVE 0x20000U
#define BITFIELD_IS_MOVING 0x40000U
//AI behaviour of particles
#define BITFIELD_TYPE_BITS_MASK 0x300000U
#define BITFIELD_TYPE_RAIN 0x0U
#define BITFIELD_TYPE_FIREFLIES 0x100000U
#define BITFIELD_TYPE_FIREFLIES_BOIDS 0x200000U
//Used to render particles with different orientations
#define BITFIELD_BILLBOARD_MODE_BITS_MASK 0x1800000U
#define BITFIELD_BILLBOARD_MODE_SCREEN_ALIGNED 0x0U
#define BITFIELD_BILLBOARD_MODE_VELOCITY_ORIENTED 0x800000U
//Lighting flags: A particle can emit light, cast shadows,
//or not affect the lighting of the scene.
#define BITFIELD_LIGHTING_MODE_BITS_MASK 0x6000000U
#define BITFIELD_LIGHTING_MODE_NONE 0x0U
#define BITFIELD_LIGHTING_MODE_LIGHT 0x2000000U
#define BITFIELD_LIGHTING_MODE_LIGHTNSHADOW 0x4000000U
#define BITFIELD_COLLIDE_WITH_DEPTH_BUFFER 0x400000U
#define BITFIELD_LIGHT_CULLED 0x8000000U
```

For both particle buffer and bitfield buffer are divided into three main section. Section 1 contains particles that emit light and cast shadows, section 2 contains particles that only emit light and section 3 is for everything else. We further divide these 3 section into visible/active and one for inactive/invisible particles. We have an additional buffer that stores the bounds of each section in a small read/write buffer that computes when the preparation shaders. 

This boundary buffer is a very DOD thing, it avoid having a `bool isAlice` in particle itself.

| Light + shadow     | Light only         | Standard           |
| ------------------ | ------------------ | ------------------ |
| Active \| Inactive | Active \| Inactive | Active \| Inactive |
So we have 3 buffers bitfield, boundary/Active and particle buffers. All tracking states.
##### Packing
The book wants to reduce how many reads the shader has to do as much as possible so it applies aggressive data-packing techniques. Packing mostly applied to the particles, and the particle sets store 32-bit precision data that can be used to unpack a given particle. 

The particle sets are not stored anywhere but returned by a `getParticleSet` function every time it's called. It's not an intensive operation  since it's mostly consists of assignments sets. 

```c++
//This assumes ParticleSet struct.
//This also assumes the bitField defines.
ParticleSet getParticleSet(uint index)
{
	const float steering = 0.02f;
	ParticleSet swarm;
	swarm.size = vec4(3.f, 3.f, 3.f, 0.003f);
	swarm.particleType = BITFIELD_TYPE_FIREFLIES_BOIDS;
	swarm.colour = vec3(1.f, 0.9f, 0.f);
	swarm.maxParticles = 1000000;
	swarm.particlePerSecond = 10000;
	//pack2vec2(steering, 2.f); is a function that unpacks uvec2 into floats.
	swarm.steeringStrengthMaxSpeed = pack2Float(steering, 2.f);
	swarm.initialAge = 10.f;
	swarm.lightRadius = LIGHT_SIZE;
	
	ParticleSet lightSet = swarm;
	lightSet.particleType = BITFIELD_TYPE_FIREFLIES;
	lightSet.lightBitfield = BITFIELD_TYPE_MODE_LIGHT;
	lightSet.maxParticles = 10000;
	lightSet.particlesPerSEcond = 1000;
	lightSet.steeringStrengthMaxSpeed = pack2Float(steering, 0.05f);
	lightSet.lightRadius = LIGHT_SIZE;
	
	ParticleSet shadow = lightSet;
	
	switch(index)
	{
		case 0:
			swarm.position = vec3(6.f, 6.f, 10.f);
			return swarm;
		case 1:
			swarm.position = vec3(-6.f, 6.f, 10.f);
			return swarm;
		case 2:
			lightSet.position = vec3(2.f, 8.f, 5.f);
			lightSet.size = vec4(10.f, 6.f, 8.f, 0.02f);
			return lightSet;
		default:
			shadow.colour = vec3(0.8f, 1.f, 0.1f);
			shadow.lightBitfield = BITFIELD_LIGHTING_MODE_LIGHTENSHADOW;
			shadow.maxParticles = 8;
			shadow.particlesPerSecond = 0.8f;
			shadow.initialAge = 10.f;
			shadow.size = vec4(10.f, 6.f, 8.f, 0.1f);
			shadow.steeringStrengthMaxSpeed = pack2vec2(steering, 1.f);
			return shadow;
	}
}
```

When unpacking this consists of normalising the value between $[0, 1)$ range and having a minimum precision. 

The current age of a particle is divided by it's `particleSet.initialAge` and the `particle.velocityAndAge`. All particles positions are relative to the particle set, before normalising the position of the particle we divide by the size of the particleSet.

```c++
ParticleData packParticle(ParticleSet particleSet, vec3 position, vec4 velocityAndAge)
{
	ParticleData data;
	position -= particleSet.position;
	position /= PARTICLE_PACKING_SCALE;
	velocityAndAge /= particleSet.maxSpeed;
	
	
	data.position = packVec3FixedPoint(position);
	data.velocityAndAge = uvec2(pack2Float(velocityAndAge.x, velocityAndAge.y), pack2Float(velocityAndAge.z, velocityAndAge.w / particleSet.initialAge));
	return data;
}
```

We want to take a particle and allow it to go out of the bounds of particle set. To do this we want to scale the particle by the particle set size depending on the particles behaviour. 

The whole point of packing particles is so we can render more particles but have less speed for a given particle. 

When we are parcking age velocity using `uvec2(pack2Float(), pack2Float())` it's coverted to a `float16_t` packed into two unsigned integers resulting in 8 bytes. If you keep all your particles within the range of $[0, 1)$ you have greater precision because the exponent is small. This allows for floading point numbers to be around $\pm0.0005$ if numbers are kept in the $[0, 1)$. 

Doing this allows to have 32 bits of precision that can be packed into three floats with values in the range of $[0, 1]$ range. We can only get a speed of 0.03 m/s^2 and same for velocity 0.03 m/s at 60FPS. This would leave us with 16bits of extra precision to store data per-particle. Since we do not need those extra bits, we pack the position into three fixed-points values of 21 bits each: 1 bit for the sign and 20 for the fractional part. The allows for a minimum epsilon of around $\frac{1}{2^{20}} = 0.0000009537$ which plenty to supoort low velocities. 

You need to be careful with the particle set if it's too big and normalisation causes aliasing or lose of precision it can be split into multiple sets with the same settings. This can allow for particle sets  to be culled more easily.

Here are packing functions
```c++
vec2 packFloat3FixedPoint(vec3 v)
{
	const float maxVal = (1  << 20) - 1;
	vec3 vUint = vec3(0.f, 0.f, 0.f);
	
	for(uint i = 0; i < 3; ++i)
	{
		//Abs number for number after the point.
		vUint[i] = uint(round(abs(v[i] * maxVal)));
		//Sign
		vUint[i] |= (v[i] > 0 ? (1 << 20) : 0);
	}
	
	//Layout of the output: [(21 bits of X - 11 bits of Y), (unused bit, remaining 10 bits of Y, 21 bits of Z)]
	return uint2((vUint.x << 11) | (vUint.y >> 10), ((vUint.y & ((1 << 10) − 1)) << 21) | vUint.z);
}
```

```c++
vec3 unpackFloat3FixedPoint(uvec2 v)
{
	const float maxVal = (1  << 20) - 1;
	vec3 vUint = vec3(0.f, 0.f, 0.f);
	vUint[0] = v.x >> 11;
	vUint[1] = ((v.x & (1 << 11) - 1) << 10) | (v.y >> 21);
	vUint[2] = (v.y & (1 << 21) - 1);
	
	//Rebuild the number: value * sign.
	vec3 data;
	for(uint i = 0; i < 3; ++i)
	{
		data[i] = (float(vUint[i] & ((1 << 20) - 1 )) / maxVal) * (vUint[i] >> 20) > 0 ? 1 : -1;
	}
	
	return data;
}
```

##### Workflow
The book has worked out that memory bandwidth is the biggest bottleneck in modern rendering approaches with particles, that's why we pack data. The simplest way to reduce bandwidth is to process things in one run.

The particle system goes through three different steps to complete a frame.
1) The prep phase. This only access particle sets. It produces the data need that needs to be to sort the particle by visibility and to compute the bounds of each section of the buffers.
2) The simulation phase. This step is used for initalise particles, kill or respawn, sort each section of the buffer, performas part of the light culling, rastersises small particles, and update the particle velocity, age and position.