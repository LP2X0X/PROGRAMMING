---
tags:
 - csharp
 - oop
 - keyword
---

# System.Object

- Every type in C# — classes, structs, enums, delegates — ultimately derives from `System.Object`. When you build a class that does not explicitly define its parent, the compiler automatically derives your type from `Object`.
- This means **every object** in .NET has a guaranteed minimum set of functionality.

```csharp
// These two declarations are identical:
class Person { }
class Person : System.Object { }
```

---

## The Full Definition

```csharp
public class Object
{
    // Virtual members (can be overridden)
    public virtual bool Equals(object obj);
    protected virtual void Finalize();
    public virtual int GetHashCode();
    public virtual string ToString();

    // Instance-level, non-virtual members (cannot be overridden)
    public Type GetType();
    protected object MemberwiseClone();

    // Static members (called on the class, not an instance)
    public static bool Equals(object objA, object objB);
    public static bool ReferenceEquals(object objA, object objB);
}
```

```ad-note
The four **virtual** members (`Equals`, `Finalize`, `GetHashCode`, `ToString`) are the ones you are expected to override in your own classes when appropriate. The non-virtual members provide fixed runtime behavior that should not change.
```

---

## Member-by-Member Breakdown

### Virtual Members

#### `ToString()`

- By default, returns the **fully qualified type name** (`Namespace.ClassName`).
- You should override this to return a meaningful string representation of the object's state.
- Extremely useful for debugging, logging, and display.

```csharp
class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int Age { get; set; }

    // Default ToString() would return "MyNamespace.Person"
    // Override to show actual state:
    public override string ToString()
        => $"[First Name: {FirstName}; Last Name: {LastName}; Age: {Age}]";
}

// Usage:
var p = new Person { FirstName = "John", LastName = "Doe", Age = 30 };
Console.WriteLine(p); // [First Name: John; Last Name: Doe; Age: 30]
```

```ad-tip
A recommended convention is to separate each name-value pair with semicolons and wrap the entire string within square brackets. Many types in the .NET base class libraries follow this approach.
```

#### `Equals(object obj)`

- By default, returns `true` only if the two objects being compared **refer to the same instance in memory** (reference equality).
- Override this to implement **value-based semantics** — returning `true` when two distinct objects have the same internal state.

```ad-important
If you override `Equals()`, you **must** also override `GetHashCode()`. This is required because hash-based collections (`Dictionary`, `HashSet`, `Hashtable`) rely on both methods working consistently. Two objects that are equal must return the same hash code.
```

**Approach 1: Field-by-field comparison**

```csharp
public override bool Equals(object obj)
{
    if (obj is not Person temp)
    {
        return false;
    }
    return temp.FirstName == this.FirstName
        && temp.LastName == this.LastName
        && temp.Age == this.Age;
}
```

**Approach 2: Using `ToString()` as a shortcut**

- If your `ToString()` accounts for all relevant field data, you can simplify `Equals()`:

```csharp
public override bool Equals(object obj)
    => obj?.ToString() == ToString();
```

```ad-warning
The `ToString()` shortcut is convenient but fragile — if `ToString()` does not include all fields that define equality, or if two different types can produce the same string, this will give wrong results. Field-by-field comparison is safer for production code.
```

#### `GetHashCode()`

- Returns an `int` that represents the object's current state.
- By default, uses the object's **memory location** to produce the hash — so two objects with identical data will return different hash codes.
- Override this whenever you override `Equals()`.

**Rules for a correct `GetHashCode()`:**
1. If `a.Equals(b)` is `true`, then `a.GetHashCode() == b.GetHashCode()` must also be `true`.
2. The reverse is NOT required — two unequal objects *may* have the same hash code (hash collision).
3. `GetHashCode()` should be **fast** and should not throw exceptions.
4. The hash code should be based on the **same fields** used in `Equals()`.

```csharp
public override int GetHashCode()
    => HashCode.Combine(FirstName, LastName, Age);
```

```ad-note
`HashCode.Combine()` (introduced in .NET Core 2.1) is the modern way to implement `GetHashCode()`. It handles null values and produces well-distributed hash codes. Before this, developers typically used XOR or prime-number multiplication manually.
```

**How hash-based collections use these methods:**

```
Dictionary.Get(key)
    1. Call key.GetHashCode()  →  find the bucket (general location)
    2. Call key.Equals(other)  →  find the exact match within the bucket
```

This is why both methods must be consistent — `GetHashCode()` narrows the search, `Equals()` confirms the match.

#### `Finalize()`

- Called by the **garbage collector** before the object's memory is reclaimed.
- You do NOT call this method directly. The GC calls it automatically.
- In C#, you write a **destructor** (`~ClassName`) rather than overriding `Finalize()` directly — the compiler translates it.

```csharp
class ResourceHolder
{
    ~ResourceHolder()
    {
        // cleanup code — the compiler turns this into a Finalize() override
        // Release unmanaged resources here
    }
}
```

```ad-important
In modern .NET, you should almost never use finalizers. Use the `IDisposable` pattern with `using` statements instead. Finalizers add overhead because the GC must run them on a separate thread and the object survives an extra GC cycle.
```

---

### Non-Virtual Instance Members

#### `GetType()`

- Returns a `Type` object that fully describes the object's type at **runtime** (Runtime Type Identification / RTTI).
- Cannot be overridden — this guarantees you always get the true type, even through polymorphism.
- The `Type` object gives you access to all metadata: methods, properties, fields, interfaces, base class, attributes, etc.

```csharp
Person p = new Person();
Type t = p.GetType();

Console.WriteLine(t.Name);           // "Person"
Console.WriteLine(t.Namespace);      // "MyNamespace"
Console.WriteLine(t.BaseType.Name);  // "Object"

// List all public methods
foreach (var method in t.GetMethods())
{
    Console.WriteLine(method.Name);
}
```

```ad-note
`GetType()` works on the **actual runtime type** of the object. So if `Employee` derives from `Person`, calling `GetType()` on an `Employee` stored in a `Person` variable returns `Employee`, not `Person`. This is the key difference from the `typeof` operator, which works on compile-time types.
```

#### `MemberwiseClone()`

- Returns a **shallow copy** of the current object — a new instance where all fields are copied bit-by-bit.
- Marked `protected`, so only the class itself (or derived classes) can call it.
- **Shallow copy** means: value-type fields are duplicated, but reference-type fields still point to the same objects.

```csharp
class Person
{
    public string Name { get; set; }
    public Address HomeAddress { get; set; } // reference type

    public Person ShallowCopy()
        => (Person)this.MemberwiseClone();
}

var original = new Person
{
    Name = "Alice",
    HomeAddress = new Address { City = "Seattle" }
};

var clone = original.ShallowCopy();

clone.Name = "Bob";
// original.Name is still "Alice" (string is immutable, assigned a new reference)

clone.HomeAddress.City = "Portland";
// original.HomeAddress.City is now ALSO "Portland" — they share the same Address object
```

```ad-warning
For a **deep copy** (where reference-type fields are also duplicated), you must implement your own clone logic — typically via the `ICloneable` interface, a copy constructor, or serialization.
```

---

### Static Members

#### `Object.Equals(object objA, object objB)`

- A null-safe way to check equality between two objects.
- Calls `objA.Equals(objB)` internally, but handles the case where `objA` is `null` without throwing.

```csharp
Object.Equals(null, null);       // true
Object.Equals(null, "hello");    // false
Object.Equals("hello", "hello"); // true (calls string.Equals)
```

#### `Object.ReferenceEquals(object objA, object objB)`

- Returns `true` only if both arguments point to the **exact same object in memory** (or both are `null`).
- Cannot be overridden — always checks reference identity, never value equality.
- Useful when you need to verify identity regardless of how `Equals()` has been overridden.

```csharp
var a = new Person { Name = "Alice" };
var b = new Person { Name = "Alice" };
var c = a;

Object.ReferenceEquals(a, b); // false — different instances
Object.ReferenceEquals(a, c); // true  — same instance
Object.ReferenceEquals(a, null); // false
```

---

## How Value Types (`struct`) Differ

- `System.ValueType` (the base of all structs) **overrides** `Equals()` and `GetHashCode()` so that they work with value-based semantics by default.
- This means two structs with the same field values are equal out of the box, unlike classes.

```csharp
struct Point
{
    public int X;
    public int Y;
}

var p1 = new Point { X = 1, Y = 2 };
var p2 = new Point { X = 1, Y = 2 };

p1.Equals(p2); // true — ValueType.Equals does field-by-field comparison
```

```ad-note
The default `ValueType.Equals()` uses **reflection** to compare fields, which is slow. For performance-critical structs, override `Equals()` and `GetHashCode()` manually.
```

---

## Summary: When to Override What

| Method | Override when... |
|---|---|
| `ToString()` | Almost always — default output (`Namespace.Type`) is rarely useful |
| `Equals()` | You need value-based equality (two objects with the same data should be equal) |
| `GetHashCode()` | **Always** when you override `Equals()` — required for hash-based collections |
| `Finalize()` | Almost never — use `IDisposable` instead |
