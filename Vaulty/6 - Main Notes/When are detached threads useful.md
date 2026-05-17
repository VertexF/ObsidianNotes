2026-05-16 08:20
Status: #baby 
Tags: [[threading]]
# When are detached threads useful

Detach means that the thread runs in the background and there is no way of communicating with it, meaning that you can't wait for it to be completed. The ownership actually is passed over to the C++ runtime library.

This threads are meant to do tasks that more or less happen across the life time of the program, like clearing up unused caches, optimising data structures. If you have another way of knowing when the threads works has been completed, then you can use it there as well as a fire-and-forget type deal.

You can also use `joinable()` to check if you can `detach()` a thread too.

Lets say you have a word process document were each tab has it's own thread. When you create a new tab to work on a new page. It makes sense to spawn a thread that doesn't need to wait for the others to finish. This would be perfect for a detached thread.

```c++
void editDocument(std::string const& filename)
{
	openDocumentAndDisplayGUI(filename);
	while(doneEditing() == false)
	{
		userCommand cmd = getUserInput();
		
		if(cmd.type == openNewDocument)
		{
			std::string const newName = getFilenameFromUser();
			std::thread t(editDocument, newName);
			t.detach();
		}
		else
		{
			processUserInput(cmd);
		}
	}
}
```

Here is a good example of when to pass an argument to std::thread. You can do this member variables too but this is certainly easier.
# References
##### Main Notes
[[Potential issues with detaching a thread]]
#### Source Notes
[[C++ Concurrency - Managing Threads]]