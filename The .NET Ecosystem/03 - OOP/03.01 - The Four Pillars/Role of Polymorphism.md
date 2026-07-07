---
tags:
 - csharp
 - oop
 - polymorphism
---

# Role of Polymorphism

- Polymorphism is the pillar of OOP that captures a language’s ability to **treat related objects in a similar manner**. It allows a base class to define a set of members (formally termed the **polymorphic interface**) that are available to all descendants.
- The word means "many forms" — the **same method call** produces **different behavior** depending on the actual type of the object at runtime.

---

## Why Polymorphism Exists

1. **Write code once, handle many types** — a method that accepts `Shape` works for `Circle`, `Rectangle`, `Triangle`, and any future shape without modification.
2. **Open/Closed Principle** — your code is open for extension (add new derived types) but closed for modification (existing code doesn’t change).
3. **Eliminates type-checking logic** — no need for long `if`/`switch` chains checking the type.

```csharp
// Without polymorphism — brittle, must modify for every new type
void DrawShape(object shape)
{
    if (shape is Circle c) c.DrawCircle();
    else if (shape is Rectangle r) r.DrawRectangle();
    else if (shape is Triangle t) t.DrawTriangle();
    // must add a new branch for every new shape...
}

// With polymorphism — clean, extensible
void DrawShape(Shape shape)
{
    shape.Draw();  // correct Draw() is called based on actual type
    // adding a new shape type requires ZERO changes here
}
```

---

## The Polymorphic Interface

A class’s polymorphic interface is constructed using **virtual** or **abstract** members. These are the two mechanisms that enable polymorphism:

### `virtual` — Default Implementation That Can Be Overridden

A `virtual` member provides a default implementation in the base class. Derived classes **may** override it, but don’t have to.

```csharp
class Employee
{
    public string Name { get; set; }
    public virtual decimal CalculateBonus() => 1000m;  // default for all employees
}

class Manager : Employee
{
    public override decimal CalculateBonus() => 5000m;  // managers get more
}

class Intern : Employee
{
    // doesn’t override — gets the default 1000
}

Employee e1 = new Manager { Name = "Alice" };
Employee e2 = new Intern { Name = "Bob" };

e1.CalculateBonus();  // 5000 — Manager’s override
e2.CalculateBonus();  // 1000 — Employee’s default
```

### `abstract` — No Default, Must Be Overridden

An `abstract` member provides **only a signature** — no implementation. Derived classes **must** override it. Abstract members can only exist in abstract classes.

```csharp
abstract class Shape
{
    public string Color { get; set; }
    public abstract double GetArea();   // no body — each shape MUST provide its own
}

class Circle : Shape
{
    public double Radius { get; set; }
    public override double GetArea() => Math.PI * Radius * Radius;
}

class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }
    public override double GetArea() => Width * Height;
}

// Shape s = new Shape();  // compile error — cannot instantiate abstract class
```

### `virtual` vs `abstract`

| | `virtual` | `abstract` |
|---|---|---|
| Has default implementation? | Yes | No |
| Override required? | No — optional | Yes — must override |
| Can exist in concrete class? | Yes | No — only in `abstract` class |
| Use when | Most derived types share the default, some customize | Every derived type must provide its own logic |

---

## How It Works at Runtime

The key insight: the **actual object’s type** determines which method runs, not the **variable’s declared type**. This is called **dynamic dispatch**.

```csharp
Employee e = new Manager("Alice", 5000);  // declared as Employee, actual type is Manager

e.CalculateBonus();  // calls Manager.CalculateBonus(), NOT Employee.CalculateBonus()
```

```
// At compile time:
//   e is declared as Employee → compiler checks Employee has CalculateBonus() ✓

// At runtime:
//   e actually points to a Manager object
//   CLR looks at the object’s type → Manager
//   Manager has an override for CalculateBonus() → calls that one
```

This is why polymorphism only works with `virtual`/`override`. Without `virtual`, the compiler uses the **declared type** at compile time (static dispatch):

```csharp
class Employee
{
    public string GetRole() => "Employee";  // not virtual
}

class Manager : Employee
{
    public new string GetRole() => "Manager";  // hiding, not overriding
}

Employee e = new Manager();
e.GetRole();   // "Employee" — static dispatch, uses declared type
               // polymorphism is BROKEN with member hiding
```

```ad-warning
Member hiding (`new`) breaks polymorphism. Always prefer `virtual` + `override` unless you have a very specific reason to hide.
```

---

## Polymorphism in Practice — Processing Collections of Base Types

The most common use of polymorphism is iterating over a collection of base-type references where each object may be a different derived type:

```csharp
List<Shape> shapes = new List<Shape>
{
    new Circle { Color = "Red", Radius = 5 },
    new Rectangle { Color = "Blue", Width = 4, Height = 6 },
    new Circle { Color = "Green", Radius = 3 }
};

foreach (Shape s in shapes)
{
    Console.WriteLine($"{s.Color}: area = {s.GetArea():F2}");
}
// Red: area = 78.54
// Blue: area = 24.00
// Green: area = 28.27

// Each call to GetArea() runs the correct version for that object’s type
// No if/switch, no casting — just polymorphism
```

---

## `System.Object` — Polymorphism Built Into .NET

Every class inherits from `System.Object`, which defines virtual methods that you can override. This is polymorphism at the framework level:

| Method | Default behavior | Override to |
|---|---|---|
| `ToString()` | Returns type name | Return meaningful string |
| `Equals(object)` | Reference equality | Compare by data |
| `GetHashCode()` | Based on reference | Support dictionaries/hashsets |

```csharp
class Person
{
    public string Name { get; set; }

    public override string ToString() => Name;
    public override bool Equals(object obj) => obj is Person p && p.Name == Name;
    public override int GetHashCode() => Name.GetHashCode();
}

Person p = new Person { Name = "Alice" };
Console.WriteLine(p);  // "Alice" — Console.WriteLine calls ToString() polymorphically
```

---

## Polymorphism vs the Other Pillars

| Pillar | Focus | Relationship to polymorphism |
|---|---|---|
| **Encapsulation** | Hide internals, protect state | Polymorphism doesn’t care *how* a method works internally, only that it exists |
| **Inheritance** | Reuse and extend behavior | Provides the class hierarchy that polymorphism operates on |
| **Abstraction** | Simplify the surface | Abstract classes/interfaces define the polymorphic contract |
| **Polymorphism** | Same call, different behavior | Relies on all three above to work |

```ad-tip
Inheritance sets up the relationship. Abstraction defines the contract. Polymorphism makes the contract come alive at runtime.
```