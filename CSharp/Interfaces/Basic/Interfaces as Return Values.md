- Interfaces can also be used as method return values. For example, you could write a method that takes an array of Shape objects and returns a reference to the first item that supports IPointy.

```csharp
// This method returns the first object in the  
// array that implements IPointy.  
static IPointy FindFirstPointyShape(Shape[] shapes)  
{  
	foreach (Shape s in shapes)  
	{  
		if (s is IPointy ip)  
		{  
			return ip;  
		}  
	}  
	return null;  
}
```