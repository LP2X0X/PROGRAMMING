- When you are handling a managed object that implements IDisposable, it is quite common to make use of structured exception handling to ensure the type’s Dispose() method is called in the event of a runtime exception.
- While this is a fine example of defensive programming, the truth of the matter is that few developers are thrilled by the prospects of wrapping every disposable type within a try/finally block just to ensure the Dispose() method is called. To achieve the same result in a much less obtrusive manner, C# supports a special bit of syntax that looks like this:
```csharp
Console.WriteLine("***** Fun with Dispose *****\n");  
// Dispose() is called automatically when the using scope exits.  
using(MyResourceWrapper rw = new MyResourceWrapper())  
{  
	// Use rw object.  
}
```

```ad-warning
If you attempt to “use” an object that does not implement IDisposable, you will receive a compiler error.
```
- When you are working with types that support the IDisposable interface, this syntactical construct will ensure that the object “being used” will automatically have its Dispose() method called once the using block has exited.
- Also, be aware that it is possible to declare multiple objects of the same type within a using scope. As you would expect, the compiler will inject code to call Dispose() on each declared object.  

```csharp
// Use a comma-delimited list to declare multiple objects to dispose.  
using(MyResourceWrapper rw = new MyResourceWrapper(), rw2 = new MyResourceWrapper())  
{  
	// Use rw and rw2 objects.  
}
```
