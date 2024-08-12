- As you saw in the previous example, allocating a chunk of memory within an unsafe context may be facilitated via the stackalloc keyword. By the very nature of this operation, the allocated memory is cleaned up as soon as the allocating method has returned (as the memory is acquired from the stack). However, assume a more complex example. During our examination of the -> operator, you created a value type named Point. Like all value types, the allocated memory is popped off the stack once the executing scope has terminated. For the sake of argument, assume Point was instead defined as a reference type, like so:

```csharp
class PointRef // <= Renamed and retyped.  
{  
	public int x;  
	public int y;  
	public override string ToString() => $"({x}, {y})";  
}
```

- As you are aware, if the caller declares a variable of type Point, the memory is allocated on the garbage-collected heap. The burning question then becomes “What if an unsafe context wants to interact with this object (or any object on the heap)?” Given that garbage collection can occur at any moment, imagine the problems encountered when accessing the members of Point at the very point in time a sweep of the heap is underway. Theoretically, it is possible that the unsafe context is attempting to interact with a member that is no longer accessible or has been repositioned on the heap after surviving a generational sweep (which is an obvious problem).
- To lock a reference type variable in memory from an unsafe context, C# provides the fixed keyword. The fixed statement sets a pointer to a managed type and “pins” that variable during the execution of the code. Without fixed, pointers to managed variables would be of little use, since garbage collection could relocate the variables unpredictably. (In fact, the C# compiler will not allow you to set a pointer to a managed variable except in a fixed statement.)
- Thus, if you create a PointRef object and want to interact with its members, you must write the following code (or receive a compiler error):

```csharp
unsafe static void UseAndPinPoint()  
{  
	PointRef pt = new PointRef  
	{  
		x = 5,  
		y = 6  
	};  

	// Pin pt in place so it will not  
	// be moved or GC-ed.  
	fixed (int* p = &pt.x)  
	{  
		// Use int* variable here!  
	}  

	// pt is now unpinned, and ready to be GC-ed once  
	// the method completes.  
	Console.WriteLine ("Point is: {0}", pt);  
}
```

- In a nutshell, the fixed keyword allows you to build a statement that locks a reference variable in memory, such that its address remains constant for the duration of the statement (or scope block). Any time you interact with a reference type from within the context of unsafe code, pinning the reference is a must.