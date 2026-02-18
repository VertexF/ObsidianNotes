
# Intro to Data Oriented Design for Games

# Reference https://www.youtube.com/watch?v=WwkuAqObplU
[[Entity component system bad]] 

##### What is Data oriented design
Data oriented design is the most useful on real time applications, basically this happens when your application has a time limit on when it should finish. When you're programming software in these enviroments it's important to think about performance on the CPU as well as the GPU. This is what Data oriented design is attempting to help you solve, it stats that

It's important to note that OOP will tell you that you should model your code around the real-world but for our cases, this isn't useful at all. Think about how a chair could be represented in a game we could have a static chair, a destructable chair, a animated chair. These all don't really connect well onto the real world of a chair.

Do not abstruct your code a head of time to future proof of design around a problem that doesn't exist, this is how you get into speggi-code. Write the code that solve the problem you need to solve.

1) Hardware is the platform.
2) Data is more important than the code.
3) Your main responsibility is to transform the data, solve that first, not the code design.

This is a terrible example of what DoD trying to guard against.
```c#
class Enemy : GameObject
{
	public bool dead;
	public string id;
	public bool frenzy;
	public Action<string> onDeath;
	
	public void Update()
	{
		if(dead)
		{
			onDead(id);
		}
		else if(GlobalConfig.globalFrenzy || frenzy)
		{
			//handling going mad
		}
		else
		{
			//normal behaviour
		}
	}
}
```

First thing you can do here is remove this update function because it happens every single frame this wastes a tonne of time because it happens for every entity. We want to update all the entities together in bulk.

```c#
class Enemy
{
	bool dead;
	string id;
	bool frenzy;
};

//This does this all at once. Unity recommends this.
void ProcessEnemies(List<Enemy> enemies)
{
	for(int i = 0; i < enemies.count; ++i)
	{
		if(enemies[i].dead)
		{
			onDead(enemies[i].id);
		}
		else if(GlobalConfig.globalFrenzy || frenzy)
		{
			//handling going mad
		}
		else
		{
			//normal behaviour
		}
	}
}
```

Another thing we can do is remove the `onDead(enemies[i].id);` the reason you would want to get rid of this is because you want to be pull cached data when you access memory. Literally anything happen when we call this function, like accessing the network or accessing the disk. If it's possible inline function calls.

```c#
//This does this all at once. Unity recommends this.
void ProcessEnemies(List<Enemy> enemies)
{
	for(int i = 0; i < enemies.count;)
	{
		if(enemies[i].dead)
		{
			//This is harder than it looks.
			enemies.RemoveAt(i--);
		}
		else if(GlobalConfig.globalFrenzy || frenzy)
		{
			//handling going mad
			++i;
		}
		else
		{
			//normal behaviour
			++i;
		}
	}
}
```

Don't be afraid of reusing code, people in the OOP space go on about DRY but sometimes you SHOULD repeat yourself to improve performance.

Lets pull out that invariant out 

```c#
//This does this all at once. Unity recommends this.
void ProcessEnemies(List<Enemy> enemies)
{
	var globalFrenzy = GlobalConfig.globalFrenzy;
	for(int i = 0; i < enemies.count; ++i)
	{
		if(enemies[i].dead)
		{
			//This is harder than it looks.
			enemies.RemoveAt(i--);
		}
		else if(globalFrenzy || frenzy)
		{
			//handling going mad
		}
		else
		{
			//normal behaviour
		}
	}
}
```

Here we pulling out something that isn't going to change out of the loop. This is because just accessing 1 variable can be very expensive. You generally want to be look out for "tight loops" these are loops go through a lot things in sequence if we have them nested that's even better if you can pull out invariants. Image this

```c#
for(int x = 0; x < 100; ++x)
{
	for(int y = 0; y < 100; ++y)
	{
		for(int z = 0; z < 100; ++z)
		{
			if(enemies[z].dead)
			{
				//Something
			}
			else if(GlobalConfig.globalFrenzy || frenzy)
			{
				//handling going mad
			}
			else
			{
				//normal behaviour
			}
		}
	}
}
```

Then that globalFrenzy loading into memory is terrible. 

In C# changing a class to a struct can make things faster because how these are stored in array.
```c#
struct Enemy
{
	string id;
	bool frenzy;
	bool dead;
};
```

This is because they are contiguous. This is because when you store classes you are storing an array pointers, so your bouncing around memory to look for the data. Just doing this can speed up your code by 30%
##### Memory alignment

When we talk about alignement of a struct we are saying that everything is aligned in memory by the biggest thing in the struct. 

When storing memory you can't just store 1 bit in memory by itself it needs to be nicely aligned with other data. Meaning that things get padded by the alignment.

So the string here is 8 bytes container.

What you would want to is to do what we did with [[Entity component system bad]] and turn those structs into an array

```c#
enum State : byte
{
	DEAD,
	FREEZY_ALIVE,
	ALIVE,
}

struct Enemy
{
	string id;
	State state;
};
```

Finally you would want to remove that string from the struct and use a int to reduce the amount of alignment in that struct by 2.

```c#
struct Enemy
{
	uint id;
	State state;
};
```

Meaning that this struct is even smaller.