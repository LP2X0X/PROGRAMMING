In C#, an interface is a type that defines a contract of methods, properties, events, and indexers that a class or a struct can implement. Interfaces specify what members a type must provide but do not implement the functionality of those members. This allows for defining capabilities that can be shared across different types without dictating how those capabilities should be implemented.

### Key Features of Interfaces

1. **No Implementation**: Interfaces cannot contain implementation code; they only declare members.
2. **Multiple Inheritance**: A class or struct can implement multiple interfaces, allowing for a form of multiple inheritance.
3. **Polymorphism**: Interfaces provide a way to achieve polymorphism, where different types can be treated uniformly based on the interface they implement.

### Defining an Interface

To define an interface, use the `interface` keyword. Here's an example:

```csharp
public interface IMovable
{
    void Move();
    int Speed { get; set; }
}
```

### Implementing an Interface

A class or struct that implements an interface must provide implementations for all its members. Here's how a class implements the `IMovable` interface:

```csharp
public class Car : IMovable
{
    public int Speed { get; set; }

    public void Move()
    {
        Console.WriteLine($"The car is moving at {Speed} mph.");
    }
}
```

### Using Interfaces

Interfaces are useful for defining methods that can operate on any object that implements a particular interface. For example:

```csharp
public void StartMoving(IMovable movable)
{
    movable.Move();
}

// Usage
Car myCar = new Car { Speed = 60 };
StartMoving(myCar);
```

### Explicit Interface Implementation

A class can explicitly implement members of an interface. This is useful when you want to prevent access to the interface members through the class type itself:

```csharp
public class Car : IMovable
{
    int IMovable.Speed { get; set; }

    void IMovable.Move()
    {
        Console.WriteLine("The car is moving.");
    }
}

// Usage
IMovable movableCar = new Car();
movableCar.Move();
```

In this example, `Speed` and `Move` are accessible only through an `IMovable` reference.

### Interfaces and Inheritance

A class can implement multiple interfaces, and an interface can inherit from one or more interfaces:

```csharp
public interface IFlyable
{
    void Fly();
}

public interface IVehicle : IMovable, IFlyable
{
    void Stop();
}

public class Plane : IVehicle
{
    public int Speed { get; set; }

    public void Move()
    {
        Console.WriteLine("The plane is moving.");
    }

    public void Fly()
    {
        Console.WriteLine("The plane is flying.");
    }

    public void Stop()
    {
        Console.WriteLine("The plane is stopping.");
    }
}
```

### Default Interface Methods (C# 8.0 and later)

C# 8.0 introduced default interface methods, allowing interfaces to provide default implementations for members. This enables adding new methods to interfaces without breaking existing implementations:

```csharp
public interface IMovable
{
    void Move();

    // Default implementation
    void Stop()
    {
        Console.WriteLine("Default stop implementation.");
    }
}

public class Car : IMovable
{
    public void Move()
    {
        Console.WriteLine("The car is moving.");
    }
}
```

### Summary

- **Interfaces**: Define contracts with methods, properties, events, and indexers.
- **Implementation**: Classes or structs implement interfaces to provide functionality for the interface members.
- **Polymorphism**: Interfaces allow treating different types uniformly based on common capabilities.
- **Explicit Implementation**: Controls access to interface members through the implementing class.
- **Multiple Inheritance**: A type can implement multiple interfaces, and interfaces can inherit from other interfaces.
- **Default Methods**: Interfaces can have default method implementations from C# 8.0 onwards.

Interfaces are a powerful feature in C# that enable flexible and extensible code design, supporting principles like polymorphism and decoupling.