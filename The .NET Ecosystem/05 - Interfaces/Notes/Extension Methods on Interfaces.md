---
tags:
 - csharp
 - interfaces
 - extension-methods
---

Extension methods on interfaces allow you to add behavior to every type that implements the interface, without modifying the interface itself or any of its implementors.

### Why Not Just Add the Method to the Interface?

An interface defines a **contract** — it says "every implementor must provide this behavior." This is the right choice when different implementations would have different logic.

But sometimes the logic is **the same for all implementations** and can be expressed entirely using the existing interface members. In that case, adding the method to the interface:

- Forces every implementor to write identical code
- Breaks all existing implementations when you add the method
- Pollutes the contract with utility concerns

Extension methods solve this by putting derived behavior **outside** the contract.

### The Decision Rule

Ask: *Could a different implementation reasonably do this differently?*

| Answer | Where it belongs | Reason |
|--------|-----------------|--------|
| **Yes** | Interface method | It's part of the contract |
| **No** | Extension method | It's derived from the contract |

### Example

```csharp
public interface IShape
{
    double Area();
    string Name { get; }
}

public class Circle : IShape
{
    public double Radius { get; }
    public string Name => "Circle";
    public Circle(double radius) => Radius = radius;
    public double Area() => Math.PI * Radius * Radius;
}

public class Rectangle : IShape
{
    public double Width { get; }
    public double Height { get; }
    public string Name => "Rectangle";
    public Rectangle(double w, double h) { Width = w; Height = h; }
    public double Area() => Width * Height;
}
```

`Area()` belongs on the interface because each shape computes it differently. But comparison and display logic is always the same:

```csharp
public static class ShapeExtensions
{
    public static bool IsLargerThan(this IShape shape, IShape other)
    {
        return shape.Area() > other.Area();
    }

    public static string Summary(this IShape shape)
    {
        return $"{shape.Name}: area = {shape.Area():F2}";
    }
}
```

Now every `IShape` gets these methods for free:

```csharp
var circle = new Circle(5);
var rect = new Rectangle(8, 4);

circle.IsLargerThan(rect);  // true
rect.Summary();              // "Rectangle: area = 32.00"
```

### Real-World Example: LINQ

`IEnumerable<T>` defines only one contract method: `GetEnumerator()`. All of LINQ — `Where`, `Select`, `OrderBy`, `Count`, `First`, `Any`, and ~200 more — are extension methods in the `System.Linq.Enumerable` static class.

This works because:
- Every LINQ operation can be expressed using just `GetEnumerator()`
- No collection needs to implement `Where()` differently
- Adding 200+ methods to `IEnumerable<T>` would be absurd for every implementor

### Extension Methods vs Default Interface Methods (C# 8+)

C# 8 introduced default interface methods, which also add behavior without breaking implementors:

```csharp
public interface IShape
{
    double Area();
    string Name { get; }

    // default interface method
    bool IsLargerThan(IShape other) => Area() > other.Area();
}
```

Key differences:

| | Extension Methods | Default Interface Methods |
|---|---|---|
| **Overridable** | No | Yes, implementors can override |
| **Access to private members** | No, only public interface | Yes, can access other interface members |
| **Discoverability** | Requires `using` the namespace | Always available on the type |
| **Target framework** | Works everywhere | Requires .NET Core 3.0+ / .NET 5+ |

**Use default interface methods** when an implementor *might* want to override with a more efficient version (e.g., `Count()` on a collection that already tracks its length).

**Use extension methods** when the logic is truly universal and no override would ever make sense.
