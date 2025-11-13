# Reference https://nothings.org/computer/iterate.html

##### Generic iteraion

```
i = 0;
while (i < n)
{
    process(i);
    ++i;
}
```

```
cur = head;
while (cur != NULL)
{
    process(cur);
    cur = cur->next;
}
```

These two piece of code look similar but they work completely differently. The difference is that the state of the iterators. The first example holds just an index of that data structure, the other uses the structure.

Consider the how the iterator has to be

