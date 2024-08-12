- Interfaces can be arranged in an interface hierarchy. Like a class hierarchy, when an interface extends an existing interface, it inherits the abstract members defined by the parent (or parents). Prior to C# 8, derived interfaces never inherit true implementation. Rather, a derived interface simply extends its own definition with additional abstract members. In C# 8, derived interfaces inherit the default implementations as well as extend the definition and potentially add new default implementations.
- Interface hierarchies can be useful when you want to extend the functionality of an existing interface without breaking existing code bases.

---

## Interface Hierarchies with Default Implementations
- When interface hierarchies also include default implementations, downstream interfaces can choose to carry the implementation from the base interface or create a new default implementation.

```csharp
public interface IDrawable  
{  
	void Draw();  
	int TimeToDraw() => 5;  
}

Console.WriteLine("***** Simple Interface Hierarchy *****");  
...  
if (myBitmap is IAdvancedDraw iAdvDraw)  
{  
	iAdvDraw.DrawUpsideDown();  
	Console.WriteLine($"Time to draw: {iAdvDraw.TimeToDraw()}");  
}  
Console.ReadLine();
```

```ad-note
Default implementations automatically carry forward to descendant interfaces even though the instance which implement the desccendant interfaces does not have access to the default implementation.
```

- When the instance implement the method, then even if we cast the instance to any interfaces, the method **on the instance** is still get executed.