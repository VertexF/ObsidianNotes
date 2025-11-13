2025-11-11 09:23
Status: #baby 
Tags: [[C++]]
# Deleting elements out of a vector within a loop

I'm going to talk about this in the context of vectors, but any C++ container that allows us to delete something has the same concerns

The for loop statement works like this:
```
for (for-init-statement; condition; expression)
{ 
	statement;
}
```

This is equal to this:
```
for-init-statement;
while (condition)
{
	statement;
	expression;
}
```

This means that the `expression` and `condition` are run every single time. Deleting something out of a for loop is valid **ONLY IF** the checking of the size of a vector is in the `condition` slot. In the C++ standard is says "*will execute expression before re-evaluating condition*" - [C++ 11 standard](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3337.pdf) which tells us that the condition is re-evaluating every loop. 

To delete things out of a vector with a loop when you need to be **EXTREMELY** careful because if you do, the vector changes underneath the index. 

This compiles but doesn't work.
```c++
for (int i = 0; i < toDelete.size(); ++i)
{
	if (toDelete[i] == 1)
	{
		toDelete.erase(toDelete.begin() + i);
	}

	if (toDelete[i] == 0)
	{
		toDelete.erase(toDelete.begin() + i);
	}
}
```

If the data is order least to greatest the vector **SHINKS** under `i` meaning that when we execute the line 

```c++
	if (toDelete[i] == 0)
	{
		toDelete.erase(toDelete.begin() + i);
	}
```

you end up shrinking the vector down while increasing the `i` by one. This is same as `i+=2` when restarting the loop. Meaning you're **SKIPPING** over an element. You can fix this by adding a ``--i;`` **AND THEN** a `continue;` to stop the index `i` from incrementing by 1 by first adding 1 then continueing to the next iteration, but this is messy.

The safer way of doing things is when looping through and you delete an item from the vector you **DO NOT** increase `i` by 1 and you don't delete an item from the vector you **DO** increase `i`

```c++
for (int i = 0; i < toDelete.size();)
{
	if (toDelete[i] == 5)
	{
		toDelete.erase(toDelete.begin() + i);
	}
	else if (toDelete[i] == 0)
	{
		toDelete.erase(toDelete.begin() + i);
	}
	else if (toDelete[i] == 1)
	{
		toDelete.erase(toDelete.begin() + i);
	}
	else 
	{
		++i;
	}
}
```

The only way to do this is build an IF-ELSE chain and only when you've not deleting things which is the ELSE you incremented `i` by one.
# References
##### Main Notes
#### Source Notes
