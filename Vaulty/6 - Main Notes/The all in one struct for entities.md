2026-10-25 09:38
Status: #baby 
Tags: [[Entity system]]
# The all in one struct for entities

For example lets say we have an entity that has an "onFire" componenet. That when an entity gets to close it, and add the "onFire" compoenent to the entities. Here we just been a single bit of information, so why not use a single bit.

```c++
enum EntityFlags : uint8_t 
{
	ACTIVE             = 1 << 0,
	CHARACTER          = 1 << 1,
	PLAYER_CONTROLLED  = 1 << 2,
	AI_CONTROLLED      = 1 << 3,
	IS_ON_FIRE         = 1 << 4,
	HAS_BOUNDS         = 1 << 5,
	DRAW_TEXT_IN_WORLD = 1 << 6,
	RENDERABLE         = 1 << 7
};

struct Entity
{
	uint32_t id;
	Vector3 position;
	uint64_t flags;

	Bounds bounds;
	Character character;
	DrawText drawText;
	Renderable renderable;
};
```

The idea here is we use this enum flag to check if on and off stuff.

```c++
//Create entities
Array<Entity*> allEntities(allocator, 64);
for(uint32_t i = 0; i < 64; ++i)
{
	if(i == 0)
	{
		Entity player;
		player.id = i;
		player.position = {0, 0, 0};
		player.flag = ACTIVE | CHARACTER | PLAYER_CONTROLLED | HAS_BOUNDS;
		
		allEntities.push();
		allEntities[i]->id = i;	
	}
}

Array<Entity*> queryNearbyEntities(Entity currentEntity)
{
	Array<Entity*> nearby(allocate, 4);
	
	for(uint32_t i = 0; i < allEntities.size; ++i)
	{	
		if((allEntities[i]->flags & ACTIVE) == false)
		{
			continue;
		}
	
		if(currentEntity.position.x >= allEntities[i].bounds.minX &&
	    currentEntity.position.x <= allEntities[i].bounds.maxX &&
	    currentEntity.position.y >= allEntities[i].bounds.minY &&
	    currentEntity.position.y <= allEntities[i].bounds.maxY &&
	    currentEntity.position.z >= allEntities[i].bounds.minZ &&
	    currentEntity.position.z <= allEntities[i].bounds.maxZ)
		{
			nearby.push(allEntities[currentEntity.id]);
		}
	}
	
	return nearby;
}

for(uint32_t i = 0; i < allEntities.size; ++i)
{
	if((allEntities[i]->flags & ACTIVE) == false)
	{
		continue;
	}
	
	if(allEntities[i]->flags & CHARACTER)
	{
		if(allEntities[i]->flags & PLAYER_CONTROLLED)
		{
			handleUserInput(allEntities[i], deltaTime);
		}
		
		if(allEntities[i]->flags & AI_CONTROLLED)
		{
			doAI(allEntities[i], deltaTime);
		}
	}
	
	if(allEntities[i]->flags & IS_ON_FIRE)
	{
		assert(allEntities[i]->flags & HAS_BOUNDS);
		
		int index = allEntities[i]->boundIndex;
		//This is a subset of the "allEntities" not a new list of entities.
		Array<Entity*> nearbyEntity = queryNearbyEntities(allEntities[i], allBounds[index]);
		
		for(uint32_t j = 0; j < nearbyEntity.size; ++j)
		{
			//NOTE Here all I'm doing is making sure I'm not comparing pointers but that actual objects.
			if(*allEntities[i] == *nearbyEntity[j])
			{
				continue;
			}
			
			allEntities[nearbyEntity.id]->flags |= IS_ON_FIRE;
		}
	}
}
```

You typically want to start with system to begin with and pull things out if you want type safety code clarity.

This wastes memory though!! Is that problem though? Is cache locality for your game an actual problem?
# References
##### Main Notes
[[What is an ECS (Entity Compoent System)]]
#### Source Notes
[[Entity component system bad]]
