---
tags:
 - csharp
 - oop
 - inheritance
---

# Role of Inheritance

- Inheritance is one of the three pillars of OOP (alongside encapsulation and polymorphism). It lets you define a new class based on an existing one, **reusing and extending** its behavior without rewriting it.
- The existing class is the **base class** (also called superclass or parent class). The new class is the **derived class** (also called subclass or child class).
- Inheritance models an [[is-a Relationship|"is-a" relationship]] — an `Employee` **is a** `Person`, a `Dog` **is an** `Animal`. The other core relationship is [[has-a Relationship|"has-a" (composition)]], where a class *contains* another object.

---

## Why Inheritance Exists

1. **Code reuse** — common data and behavior is defined once in a base class. All derived classes inherit it automatically.
2. **Polymorphism** — you can treat derived objects through their base type, writing code that works with any subclass without knowing the specific type.
3. **Hierarchical modeling** — real-world relationships (animal → mammal → dog) map naturally to inheritance chains.

```csharp
// Without inheritance — duplicated fields in every class
class Manager  { public string Name; public int Age; public int StockOptions; }
class SalesPerson { public string Name; public int Age; public int SalesNumber; }

// With inheritance — shared fields defined once
class Employee { public string Name; public int Age; }
class Manager : Employee { public int StockOptions; }
class SalesPerson : Employee { public int SalesNumber; }
```

---

## Rules of Inheritance in C\#

### Single inheritance only

- A class can have **exactly one** direct base class. C# does not support multiple class inheritance.
- To get "multiple inheritance"-like behavior, use **interfaces** — a class can implement as many interfaces as it wants.

```csharp
// Allowed:
class Manager : Employee { }

// NOT allowed:
class Manager : Employee, Person { } // compile error

// Interfaces are fine:
class Manager : Employee, ISerializable, ICloneable { }
```

### What gets inherited

- A derived class inherits **all** members from its base class (fields, properties, methods, events).
- However, it can only **access** members based on their visibility:

| Access Modifier | Inherited? | Accessible from derived class? |
|---|---|---|
| `public` | Yes | Yes |
| `protected` | Yes | Yes (see [[The protected Keyword]]) |
| `internal` | Yes | Yes (if same assembly) |
| `protected internal` | Yes | Yes |
| `private protected` | Yes | Yes (if same assembly) |
| `private` | Yes | **No** — exists in the object but cannot be directly accessed |

```ad-note
Private members **are** inherited (they exist in the derived object's memory), you just cannot access them directly. They can still be reached through inherited public/protected methods that use them.
```

### Constructors are NOT inherited

- Constructors are not inherited. A derived class must define its own constructors.
- Use the [[Calling Base Class Constructors with the base keyword|`base` keyword]] to call a parent's constructor.

```csharp
class Employee
{
    public string Name { get; set; }
    public Employee(string name) => Name = name;
}

class Manager : Employee
{
    public int StockOptions { get; set; }

    // Must explicitly call base constructor
    public Manager(string name, int stockOptions)
        : base(name)  // calls Employee(string name)
    {
        StockOptions = stockOptions;
    }
}
```

---

## The Inheritance Chain

- Every class in C# ultimately derives from `System.Object`. If you do not specify a base class, the compiler implicitly inherits from `Object`.
- Inheritance forms a **chain**: `Manager` → `Employee` → `Object`. A `Manager` has access to members from every class up the chain.

```csharp
Manager m = new Manager("Alice", 5000);
m.ToString();       // from System.Object
m.Name;             // from Employee
m.StockOptions;     // from Manager
```

---

## Overriding and Hiding Members

### `virtual` + `override` — polymorphic override

- Mark a base method as `virtual` to allow derived classes to **replace** it with `override`.
- The overridden version is called even when accessed through a base-type reference (this is polymorphism).

```csharp
class Employee
{
    public virtual string GetRole() => "Employee";
}

class Manager : Employee
{
    public override string GetRole() => "Manager";
}

Employee e = new Manager();
e.GetRole();  // returns "Manager" — the override is called
```

### `new` — member hiding (shadowing)

- If a derived class defines a member with the same name **without** `override`, it **hides** (shadows) the base member.
- The compiler will warn you. Use the `new` keyword to acknowledge it intentionally.
- Unlike overriding, the version called depends on the **reference type**, not the object type.

```csharp
class Employee
{
    public string GetRole() => "Employee";  // not virtual
}

class Manager : Employee
{
    public new string GetRole() => "Manager";  // hides, not overrides
}

Manager m = new Manager();
m.GetRole();             // "Manager" — called through Manager reference

Employee e = m;
e.GetRole();             // "Employee" — hiding means the base version is called
```

```ad-warning
Member hiding breaks polymorphism. In almost all cases, prefer `virtual` + `override` over `new`. Hiding is usually a design mistake, not a feature to reach for.
```

### `sealed override` — stop further overriding

- A derived class can mark its override as `sealed` to prevent any further derived classes from overriding it again.

```csharp
class Employee
{
    public virtual string GetRole() => "Employee";
}

class Manager : Employee
{
    public sealed override string GetRole() => "Manager";
}

class SeniorManager : Manager
{
    // Compile error — GetRole() is sealed in Manager
    // public override string GetRole() => "Senior Manager";
}
```

---

## Sealed Classes

- A class marked `sealed` cannot be inherited from at all.
- Use this when a class is **not designed** to be extended, or for a small performance benefit (the runtime can optimize virtual calls on sealed types).

```csharp
sealed class Singleton
{
    // No class can derive from Singleton
}

// Compile error:
// class MySingleton : Singleton { }
```

```ad-note
Many .NET types are sealed — `string`, `DateTime`, `StringBuilder`, etc. The designers decided these types should not be extended.
```

---

## Abstract Classes

- A class marked `abstract` **cannot be instantiated** directly — it exists only to be a base class.
- It can contain **abstract members** (no implementation — derived classes *must* override them) and **concrete members** (with implementation — inherited as-is).

```csharp
abstract class Shape
{
    public string Color { get; set; }             // concrete — inherited
    public abstract double GetArea();              // abstract — must be overridden
    public void PrintInfo()                        // concrete — inherited
        => Console.WriteLine($"{Color} shape, area = {GetArea()}");
}

class Circle : Shape
{
    public double Radius { get; set; }
    public override double GetArea() => Math.PI * Radius * Radius;
}

// Shape s = new Shape();  // compile error — cannot instantiate abstract class
Shape s = new Circle { Color = "Red", Radius = 5 };
s.PrintInfo();  // "Red shape, area = 78.54..."
```

| | Abstract Class | Concrete Class |
|---|---|---|
| Can be instantiated? | No | Yes |
| Can have abstract members? | Yes | No |
| Can have concrete members? | Yes | Yes |
| Can be sealed? | No (contradicts its purpose) | Yes |

---

## Casting in Inheritance Hierarchies

### Implicit upcast (always safe)

- A derived type can always be assigned to a base-type variable. No cast syntax needed.

```csharp
Manager m = new Manager("Alice", 5000);
Employee e = m;   // implicit upcast — Manager is an Employee
Object o = m;     // also valid — Manager is an Object
```

### Explicit downcast (can fail)

- Going from a base type to a derived type requires a cast and can throw `InvalidCastException` if the object is not actually that type.

```csharp
Employee e = new Manager("Alice", 5000);

Manager m = (Manager)e;          // works — e is actually a Manager
SalesPerson sp = (SalesPerson)e; // throws InvalidCastException at runtime
```

### Safe casting with `is` and `as`

```csharp
// "is" — returns bool
if (e is Manager mgr)
{
    Console.WriteLine(mgr.StockOptions);  // mgr is already cast
}

// "as" — returns null instead of throwing
Manager? m = e as Manager;
if (m != null)
{
    Console.WriteLine(m.StockOptions);
}
```

```ad-tip
Prefer `is` with pattern matching (the first form above). It combines the type check and cast in one step and is the most readable option in modern C#.
```

---

## Composition vs Inheritance

- Inheritance is not always the right tool. The other way to reuse behavior is [[has-a Relationship|composition ("has-a")]].

|             | Inheritance ("Is-A")                                | Composition ("Has-A")                                                |
| ----------- | --------------------------------------------------- | -------------------------------------------------------------------- |
| Coupling    | Tight — subclass depends on base class internals    | Loose — only depends on the contained object's public API            |
| Flexibility | Fixed at compile time                               | Can swap implementations at runtime                                  |
| Reuse       | Inherits everything, even what you don't need       | Cherry-pick exactly what you need                                    |
| When to use | True type hierarchies (`Dog` really is an `Animal`) | When you need behavior from another class but aren't a subtype of it |

```ad-important
**"Favor composition over inheritance"** is a well-known design guideline. Inheritance creates rigid coupling — changes to the base class ripple into all subclasses. Use inheritance for genuine "is-a" relationships. Use composition for everything else.
```
