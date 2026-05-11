---
tags:
 - csharp
 - interfaces
---

- When you want to give your custom type the ability to return an identical copy of itself to the caller, you may implement the standard `ICloneable` interface.
- It contains a single method, `Clone()`, which returns `object`. The interface does **not** specify whether the clone should be a deep copy or a shallow copy — this ambiguity is the main criticism of the interface.

```csharp
public interface ICloneable
{
    object Clone();
}
```

---

## Why Cloning Matters

By default, assigning one reference type variable to another does **not** copy the object — it copies the **reference**. Both variables point to the same object in memory:

```csharp
Person a = new Person { Name = "John", Age = 30 };
Person b = a;

b.Name = "Jane";
Console.WriteLine(a.Name); // "Jane" — both point to the same object
```

Cloning exists to solve this: you want a **separate object** with the same values, so modifying one doesn't affect the other.

---

## Shallow Copy vs Deep Copy

This is the core distinction you need to understand before implementing `Clone()`.

### Shallow Copy

Copies the object's **immediate field values**. For value types (`int`, `double`, `bool`, `struct`), this gives you an independent copy. For reference types (objects, arrays, strings*), it copies the **reference** — both the original and the clone point to the **same** nested object.

> *`string` is a reference type but is **immutable** in C#, so sharing a reference is safe — you can treat it like a value type for cloning purposes.

```csharp
public class Person : ICloneable
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address Address { get; set; }

    public object Clone()
    {
        return this.MemberwiseClone(); // built-in shallow copy
    }
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
}
```

```csharp
Person original = new Person
{
    Name = "John",
    Age = 30,
    Address = new Address { Street = "123 Main St", City = "Anytown" }
};

Person clone = (Person)original.Clone();

clone.Name = "Jane";          // safe — string is immutable, original unaffected
clone.Age = 25;               // safe — int is a value type
clone.Address.Street = "456 Elm St"; // PROBLEM — changes original too!

Console.WriteLine(original.Address.Street); // "456 Elm St" — both share the same Address
```

**What `MemberwiseClone()` actually does under the hood:**
- Allocates a new object of the same type
- Copies every field bit-for-bit (value types get duplicated, reference types get their pointer copied)
- Does **not** call any constructor

### Deep Copy

Copies the object **and** all objects it references, recursively. The result is a completely independent object graph — no shared references.

```csharp
public class Person : ICloneable
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address Address { get; set; }

    public object Clone()
    {
        Person clone = (Person)this.MemberwiseClone();
        // manually clone the reference type field
        clone.Address = new Address
        {
            Street = this.Address.Street,
            City = this.Address.City
        };
        return clone;
    }
}
```

```csharp
Person original = new Person
{
    Name = "John",
    Age = 30,
    Address = new Address { Street = "123 Main St", City = "Anytown" }
};

Person clone = (Person)original.Clone();

clone.Address.Street = "456 Elm St";

Console.WriteLine(original.Address.Street); // "123 Main St" — independent copy
Console.WriteLine(clone.Address.Street);    // "456 Elm St"
```

### When to Use Which

| Scenario | Use |
|---|---|
| Object has only value types and strings | Shallow copy is sufficient |
| Object has reference type fields you don't want shared | Deep copy |
| Object has nested objects with their own nested objects | Deep copy (recursively) |
| Object references immutable types only | Shallow copy is safe |

---

## The Problems with ICloneable

### 1. No Contract on Depth

The interface doesn't say whether `Clone()` returns a shallow or deep copy. A caller has no way to know without reading the implementation. This makes it unreliable in generic code:

```csharp
void DoSomething(ICloneable item)
{
    var copy = item.Clone(); // shallow? deep? no way to know
}
```

### 2. Returns `object`

You always need to cast the result:

```csharp
Person clone = (Person)original.Clone(); // cast required
```

This is because `ICloneable` predates generics (it's from .NET 1.0).

### 3. Recursive Cloning is Hard

If your object graph is deep or has circular references, implementing a correct deep clone is error-prone:

```csharp
public class Node : ICloneable
{
    public string Value { get; set; }
    public Node Parent { get; set; }    // circular reference
    public List<Node> Children { get; set; }

    public object Clone()
    {
        // You need to handle:
        // - cloning each child
        // - updating Parent references to point to the NEW cloned parent
        // - avoiding infinite recursion from circular references
        // This gets complex fast.
    }
}
```

---

## Modern Alternatives

Microsoft's own design guidelines recommend **against** implementing `ICloneable` on public APIs. Here are the preferred approaches:

### 1. Typed Clone Method

Avoids the `object` return type and makes depth explicit:

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address Address { get; set; }

    public Person DeepCopy()
    {
        return new Person
        {
            Name = this.Name,
            Age = this.Age,
            Address = new Address
            {
                Street = this.Address.Street,
                City = this.Address.City
            }
        };
    }
}
```

The method name tells you exactly what kind of copy you're getting.

### 2. Copy Constructor

A constructor that takes an instance of its own type:

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public Address Address { get; set; }

    public Person() { }

    public Person(Person other)
    {
        Name = other.Name;
        Age = other.Age;
        Address = new Address(other.Address);
    }
}

public class Address
{
    public string Street { get; set; }
    public string City { get; set; }

    public Address() { }

    public Address(Address other)
    {
        Street = other.Street;
        City = other.City;
    }
}
```

```csharp
Person original = new Person { Name = "John", Age = 30, Address = new Address { Street = "123 Main St", City = "Anytown" } };
Person clone = new Person(original);
```

### 3. `record` Types (C# 9+)

Records have built-in value semantics and a `with` expression that creates a shallow copy with optional modifications:

```csharp
public record Address(string Street, string City);
public record Person(string Name, int Age, Address Address);

var original = new Person("John", 30, new Address("123 Main St", "Anytown"));
var clone = original with { Name = "Jane" };

Console.WriteLine(original); // Person { Name = John, Age = 30, Address = Address { Street = 123 Main St, City = Anytown } }
Console.WriteLine(clone);    // Person { Name = Jane, Age = 30, Address = Address { Street = 123 Main St, City = Anytown } }
```

> Note: `with` performs a **shallow copy** — nested reference types are still shared. But since records are typically used with immutable data, this is usually fine.

---

## Summary

| Approach | Pros | Cons |
|---|---|---|
| `ICloneable` | Standard interface, recognized pattern | Ambiguous depth, `object` return type, discouraged by Microsoft |
| Typed method (`DeepCopy()`) | Explicit, type-safe | No shared interface for polymorphism |
| Copy constructor | Familiar pattern, chains naturally with inheritance | Verbose for large objects |
| `record` with `with` | Clean syntax, built-in | Shallow only, C# 9+ |
