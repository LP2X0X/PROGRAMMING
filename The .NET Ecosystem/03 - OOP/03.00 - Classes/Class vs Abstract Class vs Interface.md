---
tags:
 - csharp
 - oop
 - classes
---

# Class vs Abstract Class vs Interface

- These are the three core building blocks for defining types in C#. They look similar but serve different purposes and have different rules. Understanding exactly when to use which is critical to good OOP design.

---

## At a Glance

| | Class | Abstract Class | Interface |
|---|---|---|---|
| Can be instantiated? | Yes | **No** | **No** |
| Can have implemented methods? | Yes | Yes | Yes (C# 8+ default methods) |
| Can have abstract/unimplemented members? | No | Yes | Yes (all members are abstract by default) |
| Can have fields? | Yes | Yes | **No** |
| Can have constructors? | Yes | Yes | **No** |
| Can have a destructor/finalizer? | Yes | Yes | **No** |
| Can have static members? | Yes | Yes | Yes (C# 11+ static abstract) |
| Can have access modifiers on members? | Yes | Yes | Limited (public by default, private for default impl) |
| Supports multiple inheritance? | No (single base class) | No (single base class) | **Yes** (a class can implement many interfaces) |
| Can inherit from a class? | Yes | Yes | No (interfaces extend interfaces) |
| Can implement interfaces? | Yes | Yes | Yes (interface can extend interfaces) |
| Default member accessibility | `private` | `private` | `public` |

---

## Class (Concrete Class)

- A regular class is a **complete, instantiable type**. All members must be fully implemented.
- It is the default choice when you need an object with data and behavior.

```csharp
class Car
{
    public string Make { get; set; }
    public int Speed { get; private set; }

    public void Accelerate(int amount)
    {
        Speed += amount;
    }

    public override string ToString()
        => $"{Make} going {Speed} mph";
}

var car = new Car { Make = "Toyota" };
car.Accelerate(60);
```

### When to use

- You know the full implementation and don't need to force subclasses to provide anything.
- The type will be used directly — not just as a base for other types.

---

## Abstract Class

- An abstract class is a **partially implemented base class** that cannot be instantiated directly. It exists to be inherited from.
- It can mix **concrete members** (fully implemented) with **abstract members** (no implementation — derived classes must override them).

```csharp
abstract class Vehicle
{
    // Fields — allowed
    private int _mileage;

    // Constructor — allowed (called by derived constructors)
    protected Vehicle(string make)
    {
        Make = make;
    }

    // Concrete property — inherited as-is
    public string Make { get; }

    // Concrete method — inherited as-is
    public void AddMileage(int miles) => _mileage += miles;

    // Abstract method — NO body, derived class MUST implement
    public abstract void StartEngine();

    // Virtual method — has a default implementation, derived class CAN override
    public virtual string Describe()
        => $"{Make} with {_mileage} miles";
}

class Car : Vehicle
{
    public Car(string make) : base(make) { }

    // Must implement — abstract
    public override void StartEngine()
        => Console.WriteLine("Car engine: vroom!");

    // Optional — can override the virtual method
    public override string Describe()
        => base.Describe() + " (car)";
}

// new Vehicle("Toyota");  // compile error — cannot instantiate abstract
var car = new Car("Toyota");
car.StartEngine();         // "Car engine: vroom!"
```

### Key capabilities

1. **Fields** — can hold state that is shared across the hierarchy
2. **Constructors** — can initialize that state (derived classes call them via `base(...)`)
3. **Concrete methods** — provide shared, reusable logic
4. **Abstract methods** — force derived classes to implement specific behavior
5. **Virtual methods** — provide a default that derived classes can optionally override
6. **Access modifiers** — full control (`private`, `protected`, `internal`, `public`)

### When to use

- You have a group of related classes that **share common state and behavior** (fields, constructors, implemented methods).
- You want to **force** derived classes to implement certain members while **providing** default implementations for others.
- There is a clear "is-a" relationship.

---

## Interface

- An interface is a **pure contract** — it defines what a type can do, not how it does it. Any class or struct that implements the interface must fulfill the contract.
- Traditionally (before C# 8), interfaces had **zero implementation**. Since C# 8, interfaces can have **default implementations**, but they still cannot have fields or constructors.

```csharp
interface IDriveable
{
    // Implicitly public and abstract — no implementation
    void Accelerate(int amount);
    void Brake();

    // Property — no backing field, just the signature
    int Speed { get; }

    // Default implementation (C# 8+) — optional override
    string Describe() => $"Vehicle going {Speed}";
}

interface IElectric
{
    int BatteryLevel { get; }
    void Charge();
}

// A class can implement MULTIPLE interfaces
class Tesla : IDriveable, IElectric
{
    public int Speed { get; private set; }
    public int BatteryLevel { get; private set; } = 100;

    public void Accelerate(int amount) => Speed += amount;
    public void Brake() => Speed = Math.Max(0, Speed - 20);
    public void Charge() => BatteryLevel = 100;
}
```

### What interfaces CANNOT have

| Cannot have | Why |
|---|---|
| Fields | No state — interfaces define behavior, not storage |
| Instance constructors | Cannot be instantiated, and no state to initialize |
| Destructors/finalizers | No state to clean up |

### What interfaces CAN have (modern C#)

| Can have | Since |
|---|---|
| Method signatures (abstract) | Always |
| Properties, events, indexers (signatures) | Always |
| Default method implementations | C# 8.0 |
| Static members | C# 8.0 |
| Static abstract members | C# 11 |
| Access modifiers (for default implementations) | C# 8.0 |

### When to use

- You need **multiple inheritance** — a class can only have one base class but can implement many interfaces.
- You want to define a **capability** that unrelated classes share (e.g., `IDisposable`, `IComparable`, `ISerializable`).
- You want **loose coupling** — code against the interface, swap implementations easily (dependency injection, testing).
- The implementing types are **not related** by hierarchy (a `File` and a `DatabaseConnection` are both `IDisposable` but have no common base).

---

## Deep Comparison

### State (Fields)

```csharp
// Class — can have fields
class Car { private int _fuel = 100; }

// Abstract class — can have fields
abstract class Vehicle { protected int _mileage; }

// Interface — CANNOT have fields
interface IDriveable
{
    // int _speed;  // compile error
    int Speed { get; }  // property signature only, no backing field
}
```

```ad-important
This is the biggest practical difference between an abstract class and an interface. If your shared abstraction needs to **hold state** (fields), you must use an abstract class. Interfaces can only define property signatures — the implementing class provides the backing storage.
```

### Constructors

```csharp
// Class — yes
class Car
{
    public Car(string make) { }
}

// Abstract class — yes (called by derived classes)
abstract class Vehicle
{
    protected Vehicle(string make) { }
}
class Car : Vehicle
{
    public Car(string make) : base(make) { }
}

// Interface — no constructors at all
interface IDriveable { }
```

### Multiple Inheritance

```csharp
// Single class inheritance only:
class Animal { }
class Dog : Animal { }
// class Labrador : Dog, Pet { }  // compile error if Pet is a class

// Multiple interface implementation — no limit:
interface ISwimmable { void Swim(); }
interface IFetchable { void Fetch(); }
interface ITrainable { void Train(); }

class Labrador : Dog, ISwimmable, IFetchable, ITrainable
{
    public void Swim() { }
    public void Fetch() { }
    public void Train() { }
}
```

```ad-note
This is the main reason interfaces exist. When a type needs to fulfill multiple contracts from unrelated hierarchies, interfaces are the only option. C# intentionally chose single class inheritance to avoid the **diamond problem** (ambiguity when two base classes define the same member).
```

### Access Modifiers

```csharp
// Class — full control
class Car
{
    private int _fuel;
    protected void Refuel() { }
    internal void Service() { }
    public void Drive() { }
}

// Abstract class — same full control
abstract class Vehicle
{
    private int _id;
    protected abstract void StartEngine();
    public void Honk() { }
}

// Interface — members are public by default
interface IDriveable
{
    void Drive();          // implicitly public
    // private void X();   // only allowed with a default implementation (C# 8+)

    // Default implementation can be private
    private void Log() => Console.WriteLine("logged");
    void DriveAndLog() { Drive(); Log(); }
}
```

### Default Implementations

```csharp
// Abstract class — always supported
abstract class Vehicle
{
    public virtual string Describe() => "I am a vehicle";  // default, can override
    public abstract void Start();                           // no default, must override
}

// Interface — default implementations since C# 8
interface ILoggable
{
    // Default implementation — implementing class can override, but doesn't have to
    void Log(string message)
        => Console.WriteLine($"[LOG] {message}");
}

class Service : ILoggable
{
    // Does not override Log — uses the default
}

// Gotcha: default interface methods are only accessible through the interface reference
var s = new Service();
// s.Log("test");           // compile error — not visible through the class
((ILoggable)s).Log("test"); // works — accessed through interface
```

```ad-warning
Default interface implementations have a significant limitation: they are **not accessible through the implementing class reference**, only through the interface type. This is different from abstract class methods, which are accessible through any reference. This is by design — it prevents diamond problem ambiguities — but it surprises many developers.
```

### Inheritance Chains

```csharp
// Abstract class — forms a chain (each class has exactly one parent)
abstract class Vehicle { }
abstract class MotorVehicle : Vehicle { }
class Car : MotorVehicle { }
// Car → MotorVehicle → Vehicle → Object

// Interface — can extend multiple interfaces
interface IDriveable { void Drive(); }
interface IElectric { void Charge(); }
interface IElectricVehicle : IDriveable, IElectric { }

class Tesla : IElectricVehicle
{
    public void Drive() { }
    public void Charge() { }
}
```

### Combining Abstract Class + Interfaces

- In practice, you often use **both together**. The abstract class provides shared state and logic, while interfaces define additional capabilities:

```csharp
interface IFlyable
{
    void Fly();
    int Altitude { get; }
}

interface ISwimmable
{
    void Swim();
    int Depth { get; }
}

abstract class Bird
{
    public string Species { get; }
    protected Bird(string species) => Species = species;
    public abstract void MakeSound();
}

// Duck: is a Bird (shared state/behavior), can fly, can swim
class Duck : Bird, IFlyable, ISwimmable
{
    public Duck() : base("Duck") { }
    public override void MakeSound() => Console.WriteLine("Quack!");
    public void Fly() => Console.WriteLine("Flapping wings");
    public void Swim() => Console.WriteLine("Paddling");
    public int Altitude { get; private set; }
    public int Depth { get; private set; }
}
```

---

## Decision Guide

```
Do unrelated types need to share this contract?
  └─ Yes → Interface
  └─ No ↓

Do you need shared state (fields) or constructors?
  └─ Yes → Abstract Class
  └─ No ↓

Do you need to provide some default implementation 
while forcing subclasses to implement the rest?
  └─ Yes → Abstract Class
  └─ No ↓

Is the type a complete, ready-to-use implementation?
  └─ Yes → Class
  └─ No → Interface (define the contract, implement elsewhere)
```

### Rules of Thumb

1. **Start with an interface** if you're defining a capability — it gives maximum flexibility and testability.
2. **Use an abstract class** when related types share state and logic — "is-a" relationship with common fields/constructors.
3. **Use a concrete class** when the implementation is complete and you don't expect others to extend it.
4. **Combine both** when a type hierarchy needs shared state (abstract class) AND cross-cutting capabilities (interfaces).
5. **Favor interfaces for public APIs** — they allow consumers to provide any implementation, including mocks for testing.

---

## Summary

| Feature | Class | Abstract Class | Interface |
|---|---|---|---|
| Instantiate with `new` | Yes | No | No |
| Fields | Yes | Yes | No |
| Constructors | Yes | Yes | No |
| Implemented methods | Yes | Yes | Yes (C# 8+ default) |
| Abstract methods | No | Yes | Yes (all by default) |
| Virtual methods | Yes | Yes | N/A (default methods) |
| Multiple inheritance | No | No | Yes |
| Access modifiers | Full | Full | Limited |
| Can hold state | Yes | Yes | No |
| Purpose | Complete implementation | Shared base with contract | Pure contract / capability |
| Relationship | "I am a thing" | "We are a family of things" | "I can do this" |
