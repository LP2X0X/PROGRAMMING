- The Stack\<T> class represents a collection that maintains items using a last-in, first-out manner. As you might expect, Stack\<T> defines members named Push() and Pop() to place items onto or remove items from the stack.

```csharp
static void UseGenericStack()  
{  
	Stack<Person> stackOfPeople = new();  
	stackOfPeople.Push(new Person { FirstName = "Homer", LastName = "Simpson", Age = 47 });  
	stackOfPeople.Push(new Person { FirstName = "Marge", LastName = "Simpson", Age = 45 });  
	stackOfPeople.Push(new Person { FirstName = "Lisa", LastName = "Simpson", Age = 9 });  
	// Now look at the top item, pop it, and look again.  
	Console.WriteLine("First person is: {0}", stackOfPeople.Peek());  
	Console.WriteLine("Popped off {0}", stackOfPeople.Pop());  
	Console.WriteLine("\nFirst person is: {0}", stackOfPeople.Peek());  
	Console.WriteLine("Popped off {0}", stackOfPeople.Pop());  
	Console.WriteLine("\nFirst person item is: {0}", stackOfPeople.Peek());  
	Console.WriteLine("Popped off {0}", stackOfPeople.Pop());  
	try  
	{  
		Console.WriteLine("\nnFirst person is: {0}", stackOfPeople.Peek());  
		Console.WriteLine("Popped off {0}", stackOfPeople.Pop());  
	}  
	catch (InvalidOperationException ex)
	{  
		Console.WriteLine("\nError! {0}", ex.Message);  
	}  
}
```