---
tags: 
- design
- pattern
- overview
---

## What Are Design Patterns

- **Design patterns** are reusable solutions to commonly occurring problems in software design. They are not code you copy-paste — they are templates for how to solve a problem that can be adapted to many situations.
- The concept was popularized by the **Gang of Four (GoF)** — Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides — in their 1994 book *Design Patterns: Elements of Reusable Object-Oriented Software*.
- Patterns are grouped into three categories: **[[Creational Patterns]]**, **[[Structural Patterns]]**, and **[[Behavioral Patterns]]**.

```ad-important
A pattern is not always the right answer. Applying patterns where they are not needed adds unnecessary complexity. Use a pattern when the problem it solves actually exists in your code.
```

---

## The Three Categories

- **[[Creational Patterns]]** deal with **object creation mechanisms** — they abstract the instantiation process so that the system is independent of how its objects are created, composed, and represented. Includes [[01 - Singleton|Singleton]], [[02 - Factory Method|Factory Method]], [[03 - Abstract Factory|Abstract Factory]], [[04 - Builder|Builder]], and [[05 - Prototype|Prototype]].

- **[[Structural Patterns]]** deal with **object composition** — how classes and objects are combined to form larger structures while keeping them flexible and efficient. Includes [[01 - Adapter|Adapter]], [[02 - Decorator|Decorator]], [[03 - Facade|Facade]], [[04 - Proxy|Proxy]], [[05 - Composite|Composite]], [[06 - Bridge|Bridge]], and [[07 - Flyweight|Flyweight]].

- **[[Behavioral Patterns]]** deal with **communication between objects** — how objects interact and distribute responsibility. Includes [[01 - Observer|Observer]], [[02 - Strategy|Strategy]], [[03 - Command|Command]], [[04 - State|State]], [[05 - Template Method|Template Method]], [[06 - Iterator|Iterator]], [[07 - Chain of Responsibility|Chain of Responsibility]], [[08 - Mediator|Mediator]], [[09 - Memento|Memento]], and [[10 - Visitor|Visitor]].

---

## Quick Reference Table

| Pattern | Category | One-Line Summary |
|---|---|---|
| Singleton | Creational | One instance, global access |
| Factory Method | Creational | Subclass decides which object to create |
| Abstract Factory | Creational | Create families of related objects |
| Builder | Creational | Step-by-step complex object construction |
| Prototype | Creational | Clone existing objects |
| Adapter | Structural | Convert one interface to another |
| Decorator | Structural | Add responsibilities dynamically |
| Facade | Structural | Simplified interface to a subsystem |
| Proxy | Structural | Placeholder that controls access |
| Composite | Structural | Tree structure, uniform leaf/node treatment |
| Bridge | Structural | Separate abstraction from implementation |
| Flyweight | Structural | Share objects to save memory |
| Observer | Behavioral | Notify dependents of state changes |
| Strategy | Behavioral | Swap algorithms at runtime |
| Command | Behavioral | Encapsulate request as object (undo/redo) |
| State | Behavioral | Behavior changes with internal state |
| Template Method | Behavioral | Algorithm skeleton with customizable steps |
| Iterator | Behavioral | Sequential access without exposing internals |
| Chain of Responsibility | Behavioral | Pass request along a handler chain |
| Mediator | Behavioral | Centralize complex communication |
| Memento | Behavioral | Capture and restore state |
| Visitor | Behavioral | Add operations without modifying classes |

---

```ad-tip
**How to pick the right pattern:**
1. Identify the problem (object creation? structure? communication?)
2. That narrows it to one of the three categories
3. Within the category, match the specific constraint (need one instance? need to swap algorithms? need undo?)
4. If two patterns seem to fit, check whether you actually need the pattern at all - simpler code is always better
```
