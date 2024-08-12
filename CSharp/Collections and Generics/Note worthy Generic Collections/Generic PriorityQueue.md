- Introduced in .NET 6/C# 10, the PriorityQueue works just like the Queue\<T> except that each queued item is given a priority. When items are dequeued, they are removed from lowest to highest priority.

```csharp
static void UsePriorityQueue()
{
	Console.WriteLine("* Fun with Generic Priority Queues *\n");
	
	PriorityQueue<Person, int> peopleQ = new();
	peopleQ.Enqueue(new Person { FirstName = "Lisa", LastName = "Simpson", Age = 9 }, 1);
	peopleQ.Enqueue(new Person { FirstName = "Homer", LastName = "Simpson", Age = 47 }, 3);
	peopleQ.Enqueue(new Person { FirstName = "Marge", LastName = "Simpson", Age = 45 }, 3);
	peopleQ.Enqueue(new Person { FirstName = "Bart", LastName = "Simpson", Age = 12 }, 2);
	while (peopleQ.Count > 0)
	{
		Console.WriteLine(peopleQ.Dequeue().FirstName); //Displays Lisa
		Console.WriteLine(peopleQ.Dequeue().FirstName); //Displays Bart
		Console.WriteLine(peopleQ.Dequeue().FirstName); //Displays either Marge or Homer
		Console.WriteLine(peopleQ.Dequeue().FirstName); //Displays the other priority 3 item
	}
}
```

```ad-important
If more than one item is set to the current lowest priority, the order of dequeuing is not guaranteed. As shown in code sample, the third call to Dequeue() will return either Homer or Marge, as they are both set to a priority of three. The fourth call will then return the other person. If exact order matters, you must ensure values for each priority are unique.
```