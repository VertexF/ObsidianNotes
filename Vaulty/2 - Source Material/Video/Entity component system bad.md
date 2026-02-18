# Reference https://www.youtube.com/watch?v=0S-KGLmLYnI

##### What is an ECS
In an ECS a system is something that can declare it's read and write dependencies to get some automatic parallelism. A component system is just something that can add data to an entity at run time.

Has-a is what composition is about in the OOP space it's really not important why it's called that. All it really means on a practical level is that you have classes or structs inside classes and structs

```c++
struct A{};
struct B
{
	A apple;
};
```

Apparent benefits
1) It allows for composition of functionality create emergent behaviour.
2) It's for designers because of the tooling it provides.
3) Has HUGE performant benefits.

These are promotted because of the comerical game engine space but the benefits are more of a half truth.

What's actually true when it comes to game development
1) Not all the benefits are exclusive to ECS's
2) You don't need all the features to ship a game.
3) ECS is a compression algorithm.

Compositions is a properties of an ECS not it's outcome, so if you want composition you don't need an ECS for just that.

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

As the complexity increases you can have handles to the data in your entity.

```c++
struct Entity
{
	uint32_t id;
	Vector3 position;
	uint64_t flags;
	
	int32_t boundIndex;
	int32_t characterIndex;
	int32_t renderableIndex;
	int32_t drawTextIndex;
};

Array<Entity> allEntities;
Array<Bounds> allBounds;
Array<Character> allCharacters;
Array<Renderable> allRenderables;
Array<DrawText> allDrawTexts;

Array<Entity*> allEntities(allocator, 64, 64);
Array<Entity*> allBounds(allocator, 64);
Array<Entity*> allCharacters(allocator, 4);
Array<Entity*> allRenderables(allocator, 64);
Array<Entity*> allDrawTexts(allocator, 64);
for(uint32_t i = 0; i < 64; ++i)
{
	allEntities[i]->id = i;
	
	if(i == 0)
	{
		allEntities[i]->characterIndex = i;
		allEntities[i]->boundIndex = i;
		allEntities[i]->renderableIndex = i;
		allEntities[i]->drawTextIndex = -1;
		allEntities[i]->position = {0, 0, 0};
		allCharacters.push({100, 50});
		allBounds.push({10, 10});
		allRenderables.push({true, "Assets/images/character.png"});
	}
	
	//... Set up other here.
}

```

Now what you've done is simply compressed the memory usage of the entity by using indices instead of having potentionally super large structs. All we are doing here is stripping away data we don't need, this is very DoD friendly.

Going back to our mega struct example we can simple use a union within that entity.
```c++
struct Entity
{
	Bounds bounds;
	union
	{
		Character character;
		DrawText drawText;
	};
	Renderable renderable;
};

Array<Entity*> allEntities(allocator, 64, 64);
```

In this case because we know we will never have a drawText object AND a character object this reduces the amount memory. The union will still be the size of the largest struct but any structs less than that, will cost nothing.
##### Complexity

These compressed data things cost you by increasing the complexity of your game. It's extremely important to note that you don't have as much time as yout think so you should only go out to solve problems you do have or you'll **DEFINITELY** will add more problems you'll have to solve.
##### Narrowing the complexity

Instead of having Enity to loop through all the entities you could literally just write structs that do exactly what you need, have just arrays of these entities.

For example
```c++
struct Enemy
{
	uint32_t id;
	EnemyDefinition* definition;
	float moveSpeed;
	int32_t currentHealth;
};

struct Tower
{
	ProjectileDefinition* projectileToSpawn;
	DamageType damageType;
	int32_t damage;
	float cooldown;
};

struct Projectile
{
	ProjectileDefintion *defintion;
	uint32_t targetID;
};
```

Then you literally just make an array of these and access the array at different points to then do a thing you want. 

```c++
uint8_t bossEnemyID = allEnemies[10].id;
	
//... Other stuff in your game happens here.
	
for(uint32_t i = 0; i < allTowers.size; ++i)
{
	allTowers[i].damage -= allEnemies[bossEnemyID].moveSpeed;
}
```

Do you need to make them all just "Entities" can you get around this by just use array's of struct for your entities that interact with each other like that? If you can that's a W.
##### System 

You can definite which components a system depends on to read and which components it modifies. With this you can write an algorithm to build up a dependencies graph. Meaning that if you have an entity contains a component that affects another component (say like if damage and health scale with an attack speed buff) the system will know once you update it will automatically update damage and health.

This allows for components to be updated in parallel with other components. 

This isn't as good as you think the complexity is so high because the system depenencies are so difficult to get right, and making everything generic as possible means adding new features affects everything in your entities.

This also doesn't increase the cache locality of things, that happens if your data is tightly packed in arrays in contiguously in memory.