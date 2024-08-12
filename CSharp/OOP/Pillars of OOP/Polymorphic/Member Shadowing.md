Member shadowing in C# occurs when a derived class defines a member with the same name as a member in its base class. This can lead to situations where the member in the derived class hides or "shadows" the member in the base class. Shadowing can be explicit or implicit, and it's important to understand how to manage and use it properly to avoid confusion.

### Example of Member Shadowing

Let's look at an example to illustrate member shadowing.

```csharp
using System;

class BaseClass
{
    public string Name = "Base Name";

    public void Display()
    {
        Console.WriteLine("BaseClass Display");
    }
}

class DerivedClass : BaseClass
{
    // Shadowing the field 'Name' in BaseClass
    public new string Name = "Derived Name";

    // Shadowing the method 'Display' in BaseClass
    public new void Display()
    {
        Console.WriteLine("DerivedClass Display");
    }
}

class Program
{
    static void Main()
    {
        BaseClass baseObj = new BaseClass();
        baseObj.Display(); // Output: BaseClass Display
        Console.WriteLine(baseObj.Name); // Output: Base Name

        DerivedClass derivedObj = new DerivedClass();
        derivedObj.Display(); // Output: DerivedClass Display
        Console.WriteLine(derivedObj.Name); // Output: Derived Name

        // Accessing members through base class reference
        BaseClass baseRefToDerived = new DerivedClass();
        baseRefToDerived.Display(); // Output: BaseClass Display
        Console.WriteLine(baseRefToDerived.Name); // Output: Base Name
    }
}
```

### Explanation

1. **BaseClass**:
    - Contains a field `Name` and a method `Display`.

2. **DerivedClass**:
    - Defines a new field `Name` with the `new` keyword, which shadows the `Name` field in `BaseClass`.
    - Defines a new method `Display` with the `new` keyword, which shadows the `Display` method in `BaseClass`.

3. **Program Class**:
    - Creates an instance of `BaseClass` and calls its `Display` method and `Name` field.
    - Creates an instance of `DerivedClass` and calls its `Display` method and `Name` field.
    - Creates a `BaseClass` reference to a `DerivedClass` instance and calls its `Display` method and `Name` field. Notice that the base class reference accesses the base class members, not the shadowed members in the derived class.

### Key Points

- **Using the `new` keyword**: The `new` keyword is used to explicitly indicate that a member in the derived class is intended to shadow a member in the base class.
- **Member Access**: When accessing members through a base class reference, the base class version of the member is accessed. When accessing members through a derived class reference, the derived class version is accessed.
- **Avoiding Confusion**: Member shadowing can sometimes lead to confusion, so it's generally recommended to avoid using the same member names in base and derived classes unless necessary.

### Best Practices

- **Explicitly Use `new`**: Always use the `new` keyword to indicate intentional shadowing, which makes your code clearer.
- **Prefer Overriding**: When possible, prefer using virtual methods and overriding them in derived classes, as this supports polymorphism and makes the behavior more predictable.
- **Naming Conventions**: Use different names for members in base and derived classes to avoid accidental shadowing and improve code readability.

### Example of Using Virtual and Override

Using `virtual` and `override` keywords avoids shadowing and leverages polymorphism:

```csharp
using System;

class BaseClass
{
    public virtual void Display()
    {
        Console.WriteLine("BaseClass Display");
    }
}

class DerivedClass : BaseClass
{
    public override void Display()
    {
        Console.WriteLine("DerivedClass Display");
    }
}

class Program
{
    static void Main()
    {
        BaseClass baseObj = new BaseClass();
        baseObj.Display(); // Output: BaseClass Display

        DerivedClass derivedObj = new DerivedClass();
        derivedObj.Display(); // Output: DerivedClass Display

        BaseClass baseRefToDerived = new DerivedClass();
        baseRefToDerived.Display(); // Output: DerivedClass Display
    }
}
```

In this version, the `Display` method in `BaseClass` is marked as `virtual`, and the `Display` method in `DerivedClass` is marked as `override`. This ensures that the correct method is called based on the actual object type, supporting polymorphism.