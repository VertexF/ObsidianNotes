2026-05-16 11:11
Status: #baby 
Tags: [[threading]]
# Non-const reference arguments with threads

You can't have a non-const reference like this with `std::thread`

```c++
void updateData(WidgetID id, WidgetData &data)
{
	//does stuff
}
void bad(WidgetID id)
{
	WidgetData data;
	std::thread t(updateData, w, data);
	displayResult();
	t.join();
	processWidgetData(data);
}
```

If you want to do that you need to use `std::ref` this is because the `std::thread` internals doesn't know that the function needs a reference. So it bindly **copies** the data into temporary variables and sends over an **r-value**. This is so `std::thread` only deals with `std::move` operations.

```c++
std::thread t(updateData, w, std::ref(data));
```
# References
##### Main Notes
[[Thread string arguments]]
[[Arguments that can only be moved with threads]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]