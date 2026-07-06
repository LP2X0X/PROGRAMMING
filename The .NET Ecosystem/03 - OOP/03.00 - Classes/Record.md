---
tags:
 - csharp
 - oop
 - features
---

# Records

- A **record** is a special kind of type designed for **holding data**. When you declare a record, the compiler automatically generates all the boilerplate you would normally write by hand — `Equals()`, `GetHashCode()`, `ToString()`, deconstruction, copy logic, and the `==` / `!=` operators — all with **value-based semantics**.
- Records were introduced in **C# 9** (`record` / `record class`) and extended in **C# 10** (`record struct`).

---

## The Problem Records Solve

- Most classes in a typical application are bags of data — DTOs, events, messages, configuration objects. For these types you almost always want:
  - Two objects with the same data to be **equal**
  - A useful `ToString()` output for debugging
  - An easy way to **copy with modifications** (immutability)
- Without records, you write all of this yourself:

```csharp
// Without records — all this boilerplate for a simple data type:
class Person
{
    public string Name { get; init; }
    public int Age { get; init; }

    public override bool Equals(object obj)
        => obj is Person p && p.Name == Name && p.Age == Age;

    public override int GetHashCode()
        => HashCode.Combine(Name, Age);

    public override string ToString()
        => $"Person {{ Name = {Name}, Age = {Age} }}";

    public static bool operator ==(Person a, Person b) => a.Equals(b);
    public static bool operator !=(Person a, Person b) => !a.Equals(b);

    // Plus deconstruction, copy logic...
}
```

```csharp
// With records — one line:
record Person(string Name, int Age);
```

That single line generates **everything** above.

---

## Syntax

### Positional syntax (most common)

- Parameters in the declaration become public `init`-only properties automatically:

```csharp
record Person(string Name, int Age);

// The compiler generates:
// public string Name { get; init; }
// public int Age { get; init; }
// + Equals, GetHashCode, ToString, Deconstruct, ==, !=, with-support
```

### Standard property syntax

- You can also declare a record with a body, like a regular class:

```csharp
record Person
{
    public string Name { get; init; }
    public int Age { get; init; }
    public string FullInfo => $"{Name}, age {Age}";
}
```

### Mixing both

```csharp
record Person(string Name, int Age)
{
    // Additional property not in the positional parameters
    public string Greeting => $"Hi, I'm {Name}";
}
```

---

## What You Get Automatically

| Feature | What it does |
|---|---|
| `Equals()` | Value-based — two objects with the same property values are equal |
| `GetHashCode()` | Based on all properties, consistent with `Equals` |
| `ToString()` | `Person { Name = Alice, Age = 30 }` |
| `==` and `!=` | Value-based comparison (delegates to `Equals`) |
| `Deconstruct()` | `var (name, age) = person;` |
| `with` expression | Non-destructive copy: `var p2 = p1 with { Age = 31 };` |
| `IEquatable<T>` | Implemented automatically |

### Value-Based Equality

```csharp
record Person(string Name, int Age);

var p1 = new Person("Alice", 30);
var p2 = new Person("Alice", 30);

p1 == p2;          // true  — same data
p1.Equals(p2);     // true
ReferenceEquals(p1, p2); // false — still different objects in memory
```

```ad-important
This is the fundamental difference from a regular `class`, where `==` checks if two variables point to the **same object in memory**, not whether they hold the same data.
```

### ToString()

```csharp
record Person(string Name, int Age);

var p = new Person("Alice", 30);
Console.WriteLine(p);
// Output: Person { Name = Alice, Age = 30 }
```

- Compare this to a regular class, which would print `MyNamespace.Person`. The record's `ToString()` shows all property values — extremely useful for debugging and logging.

### Deconstruction

```csharp
record Person(string Name, int Age);

var p = new Person("Alice", 30);
var (name, age) = p;  // positional deconstruction

Console.WriteLine(name); // Alice
Console.WriteLine(age);  // 30
```

```ad-note
Deconstruction is only generated for **positional** parameters (those in the parentheses of the declaration). Properties defined in the body are not included.
```

---

## The `with` Expression (Non-Destructive Mutation)

- Records are **immutable by default** (properties are `init`-only). You do not modify a record — you create a copy with changes using `with`:

```csharp
record Person(string Name, int Age);

var alice = new Person("Alice", 30);
var olderAlice = alice with { Age = 31 };

Console.WriteLine(alice);      // Person { Name = Alice, Age = 30 }  — unchanged
Console.WriteLine(olderAlice); // Person { Name = Alice, Age = 31 }  — new copy
```

- The `with` expression copies **all** properties and then applies the changes you specify. Properties you do not mention keep their original values.

```csharp
// Copy multiple properties at once:
var bob = alice with { Name = "Bob", Age = 25 };
// Person { Name = Bob, Age = 25 }
```

```ad-tip
The `with` expression is the main reason to choose records over classes for immutable data. With a regular class, you would have to write a manual copy constructor or a `Clone()` method.
```

---

## Record Types: `record` vs `record struct` vs `readonly record struct`

```csharp
record Person(string Name, int Age);                    // reference type, immutable
record struct Point(int X, int Y);                      // value type, MUTABLE
readonly record struct Point(int X, int Y);             // value type, immutable
```

| | `record` (class) | `record struct` | `readonly record struct` |
|---|---|---|---|
| Underlying type | reference (heap) | value (stack) | value (stack) |
| Equality | value-based | value-based | value-based |
| Mutable? | no (`init` properties) | **yes** (`set` properties) | no (`init` properties) |
| `with` support | yes | yes | yes |
| Can be `null`? | yes | no | no |
| Inheritance | yes | no | no |

```ad-warning
`record struct` generates **mutable** properties (`{ get; set; }`) by default, unlike `record class` which generates `init`-only properties. If you want an immutable value-type record, use `readonly record struct`.
```

### When to pick which

- **`record`** — the default choice for data objects (DTOs, events, configs). Reference type, immutable, supports inheritance.
- **`readonly record struct`** — when you need value-type semantics (small, short-lived, no heap allocation) and immutability. Good for things like `Point`, `Money`, `Color`.
- **`record struct`** — rare. Only when you need a value-type record that is intentionally mutable.

---

## Record Inheritance

- `record class` types support **inheritance** (just like regular classes). `record struct` types do not.

```csharp
record Person(string Name, int Age);
record Employee(string Name, int Age, string Department) : Person(Name, Age);

var e = new Employee("Alice", 30, "Engineering");
Console.WriteLine(e);
// Employee { Name = Alice, Age = 30, Department = Engineering }
```

### Equality and inheritance

- Record equality is **type-aware**. A `Person` and an `Employee` with the same `Name` and `Age` are **not equal**, because they are different types:

```csharp
record Person(string Name, int Age);
record Employee(string Name, int Age, string Department) : Person(Name, Age);

var person = new Person("Alice", 30);
var employee = new Employee("Alice", 30, "Engineering");

person == employee; // false — different types
```

```ad-note
This is achieved through a hidden compiler-generated `EqualityContract` property that checks the runtime type. This prevents subtle bugs where a base record would incorrectly match a derived record.
```

### Using `with` on derived records

```csharp
var alice = new Employee("Alice", 30, "Engineering");
var bob = alice with { Name = "Bob" };

// bob is still an Employee, not a Person:
Console.WriteLine(bob);
// Employee { Name = Bob, Age = 30, Department = Engineering }
```

---

## Adding Behavior to Records

- Records are not limited to just data. You can add methods, computed properties, and custom constructors:

```csharp
record Person(string FirstName, string LastName, int Age)
{
    // Computed property
    public string FullName => $"{FirstName} {LastName}";

    // Method
    public bool IsAdult() => Age >= 18;

    // Custom validation in constructor body
    public Person
    {
        if (Age < 0) throw new ArgumentOutOfRangeException(nameof(Age));
    }
}
```

```ad-note
The parameterless `public Person { }` block shown above is a **primary constructor body** — it runs after the positional properties are assigned, letting you add validation without rewriting the full constructor.
```

---

## Overriding Generated Members

- You can override any of the compiler-generated members if you need custom behavior:

```csharp
record Person(string Name, int Age)
{
    // Custom ToString
    public override string ToString()
        => $"{Name} (age {Age})";

    // Custom equality — ignore Age, compare only by Name
    public virtual bool Equals(Person other)
        => other is not null && Name == other.Name;

    public override int GetHashCode()
        => Name.GetHashCode();
}
```

```ad-warning
If you override `Equals`, you should also override `GetHashCode` for consistency — the same rule as regular classes. Be careful overriding equality on records, as it can surprise consumers who expect all properties to participate.
```

---

## Records vs Classes vs Structs — When to Use What

| Scenario | Use |
|---|---|
| Simple data carrier (DTO, event, message) | `record` |
| Value object in DDD (`Money`, `Address`) | `record` or `readonly record struct` |
| Small, short-lived data (coordinates, RGB) | `readonly record struct` |
| Mutable entity with identity and behavior | `class` |
| Lightweight value type with custom logic | `struct` |
| Need inheritance in your data types | `record` (not struct) |

### Rules of thumb

1. **If equality should be based on data** → use a record
2. **If equality should be based on identity** (an entity with an ID) → use a class
3. **If you need heap allocation and inheritance** → `record`
4. **If you need stack allocation and it's small** → `readonly record struct`
5. **If you're not sure** → start with `record`, refactor later if needed

---

## Common Patterns

### DTOs / API responses

```csharp
record WeatherForecast(DateTime Date, int TempC, string Summary);

// Deserialize from JSON, compare, log — all work out of the box
```

### Domain events

```csharp
record OrderPlaced(Guid OrderId, DateTime Timestamp, decimal Total);
record OrderShipped(Guid OrderId, DateTime Timestamp, string TrackingNumber);
```

### Configuration

```csharp
record DatabaseConfig(string Host, int Port, string DatabaseName)
{
    public string ConnectionString
        => $"Server={Host};Port={Port};Database={DatabaseName};";
}
```

### Replacing tuples with named records

```csharp
// Instead of this (unnamed, hard to read):
(string, int) GetPerson() => ("Alice", 30);

// Use this:
record Person(string Name, int Age);
Person GetPerson() => new("Alice", 30);
```
