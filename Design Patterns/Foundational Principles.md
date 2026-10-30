---
tags: [design, pattern, principles]
aliases: [Design Principles, OO Design Principles]
---

These principles underpin **all** design patterns. The GoF patterns are specific applications of these ideas — knowing the principles helps you recognize when a pattern fits and when a simpler solution is enough.

---

## 1. Encapsulate What Varies

> Take the parts that vary and encapsulate them, so that later you can alter or extend the parts that vary without affecting those that don't.

This is principle #1 in both the GoF book and *Head First Design Patterns*. Almost every pattern is an application of it:

| What varies | Pattern that encapsulates it |
|---|---|
| Algorithm / behavior | [[02 - Strategy\|Strategy]] |
| Object creation | [[02 - Factory Method\|Factory Method]], [[03 - Abstract Factory\|Abstract Factory]] |
| State transitions | [[04 - State\|State]] |
| Response to events | [[01 - Observer\|Observer]] |
| What gets executed | [[03 - Command\|Command]] |
| Traversal logic | [[06 - Iterator\|Iterator]] |
| Steps of an algorithm | [[05 - Template Method\|Template Method]] |

![[Pasted image 20261030110801.png|center]]

---

## 2. Program to an Interface, Not an Implementation

Depend on abstractions (interfaces, abstract classes), not concrete classes. This lets you swap implementations without changing the code that uses them.

```csharp
// Bad — tied to a concrete class
SqlLogger logger = new SqlLogger();

// Good — depends on abstraction
ILogger logger = new SqlLogger();  // can swap to FileLogger, ConsoleLogger, etc.
```

This is why [[02 - Strategy|Strategy]] uses an interface for the behavior, why [[02 - Factory Method|Factory Method]] returns an abstract type, and why [[01 - Observer|Observer]] notifies through an interface.

Closely related to the [[5 - Dependency Inversion Principle|Dependency Inversion Principle]] in SOLID.

---

## 3. Favor Composition Over Inheritance

Instead of inheriting behavior from a parent class, **compose** objects by giving them references to other objects that provide the behavior.

```csharp
// Inheritance — rigid, one behavior baked in
class Duck : FlyingBird { }

// Composition — flexible, behavior can change at runtime
class Duck
{
    private IFlyBehavior _flyBehavior;    // can be FlyWithWings, NoFly, RocketFly...
    
    public void PerformFly() => _flyBehavior.Fly();
    public void SetFlyBehavior(IFlyBehavior fb) => _flyBehavior = fb;
}
```

Inheritance creates an **is-a** relationship that's hard to change. Composition creates a **has-a** relationship that's easy to swap. This is the foundation of [[02 - Strategy|Strategy]], [[02 - Decorator|Decorator]], [[04 - State|State]], and [[06 - Bridge|Bridge]].

---

## 4. Strive for Loosely Coupled Designs

Objects that interact should know as little about each other as possible. When two objects are loosely coupled, they can change independently.

[[01 - Observer|Observer]] is the textbook example — the subject doesn't know what its observers do, only that they implement an interface. Adding or removing observers changes nothing about the subject.

---

## 5. The Hollywood Principle — "Don't Call Us, We'll Call You"

High-level components tell low-level components when to participate — not the other way around. The framework calls your code, you don't call the framework.

This applies to [[05 - Template Method|Template Method]] (the base class calls the subclass hooks), [[01 - Observer|Observer]] (the subject calls the observers), and dependency injection in general.

---

## How They Connect

These principles reinforce each other:

```
Encapsulate what varies
    → you need an abstraction for the varying part
    → Program to an interface
        → the abstraction is a separate object, not a parent class
        → Favor composition over inheritance
            → the objects are independent
            → Loose coupling
```

```ad-tip
If you're unsure whether to apply a pattern, ask: **"What part of this code is likely to change?"** If the answer is "nothing," you don't need a pattern. If something clearly varies, encapsulate it — and a pattern may naturally emerge.
```
