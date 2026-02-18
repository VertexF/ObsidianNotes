2026-02-11 11:05
Status: #baby 
Tags: [[performance]] [[DOD]]
# Hoisting out invariants

Pull out that invariants out of loops as this can avoid evict a cache line and avoid having to look up things in memory over and over again. If at all possible do so, this will get more powerful in tighter loops.

Bad example
```c#
void ProcessEnemies(List<Enemy> enemies)
{
	for(int i = 0; i < enemies.count; ++i)
	{
		if(GlobalConfig.globalFrenzy || frenzy)
		{
			//handling going mad
		}
	}
}
```

Good example
```c#
//This does this all at once. Unity recommends this.
void ProcessEnemies(List<Enemy> enemies)
{
	var globalFrenzy = GlobalConfig.globalFrenzy;
	for(int i = 0; i < enemies.count; ++i)
	{
		if(globalFrenzy || frenzy)
		{
			//handling going mad
		}
	}
}
```
# References
##### Main Notes
[[What is data oriented design]]
#### Source Notes
[[Intro to Data Oriented Design for Games]]