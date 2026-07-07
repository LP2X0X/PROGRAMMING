---
tags:
  - uml
  - object-diagram
  - structural
---

## 🔹 What It Shows

An object diagram is a **snapshot of instances at a specific moment in time**. Where a [[Class Diagram]] defines the blueprint, an object diagram shows what the system actually looks like when it is running -- concrete objects with real values and the links between them.

Think of it as a photograph of memory during execution. You pick one instant, freeze it, and draw every object that exists with its current attribute values and connections.

## 🔹 Object Diagram vs Class Diagram

| Aspect               | Class Diagram                        | Object Diagram                        |
| --------------------- | ------------------------------------ | ------------------------------------- |
| Shows                 | Classes (types)                      | Objects (instances)                   |
| Attributes            | `name : String`                      | `name = "John"`                       |
| Connections           | Associations (with multiplicity)     | Links (concrete connections)          |
| Multiplicity          | `1..*`, `0..1`, etc.                 | Exact count visible by counting links |
| Name format           | `ClassName`                          | `objectName : ClassName` (underlined) |
| Purpose               | Define structure                     | Validate / illustrate structure       |
| How many can exist?   | One per design                       | Many -- one per scenario              |

A class diagram says "a Customer **can** have many Orders." An object diagram says "right now, alice has **two** Orders: order42 and order43."

## 🔹 Object Notation

The basic object box uses the same two-compartment rectangle as a class, but the header is **underlined** and attributes show **concrete values** instead of types.

**Named object:**

```
┌──────────────────────────┐
│  alice : Customer         │  ← underlined in real UML
├──────────────────────────┤
│ name = "Alice Walker"     │
│ email = "alice@mail.com"  │
│ loyaltyTier = "Gold"      │
└──────────────────────────┘
```

Key rules from [[Common Notation]]:

- The header follows the pattern `objectName : ClassName` -- both parts are optional but at least one must appear.
- The name compartment is **underlined** to distinguish it from a class.
- Attribute values use `=` not `:` -- you are assigning, not declaring.
- Operations are **not shown** (objects don't redefine behavior).

**Anonymous object** -- when the identity does not matter, omit the object name:

```
┌──────────────────────────┐
│  : Customer               │  ← no object name, just the class
├──────────────────────────┤
│ name = "Bob Smith"         │
│ email = "bob@mail.com"     │
└──────────────────────────┘
```

**Orphan object** -- rare, but you can omit the class name if it is obvious:

```
┌──────────────────────────┐
│  config                    │  ← no class name
├──────────────────────────┤
│ maxRetries = 3             │
│ timeout = 5000             │
└──────────────────────────┘
```

## 🔹 Links Between Objects

Links are **instances of associations** from the [[Class Diagram]]. Where the class diagram draws an association line between two classes, the object diagram draws a **link** (a solid line) between two specific objects.

```
Class Diagram (association):

  ┌──────────┐         ┌──────────┐
  │ Customer │ 1   0..*│  Order   │
  └──────────┘─────────└──────────┘

Object Diagram (links):

  ┌─────────────────┐       ┌────────────────┐
  │ alice : Customer │──────│ order42 : Order │
  └─────────────────┘  │    └────────────────┘
                       │    ┌────────────────┐
                       └────│ order43 : Order │
                            └────────────────┘
```

| Association concept       | Object diagram equivalent                        |
| ------------------------- | ------------------------------------------------ |
| Association line          | Link (solid line, no arrowhead usually)           |
| Multiplicity `1..*`       | Visible by counting the actual links drawn        |
| Association name          | Can label the link, but often omitted             |
| Role names                | Can appear at link ends                           |
| Aggregation / Composition | Same diamond notation on the link (see [[Relationships]]) |

Links **never** show multiplicity numbers -- the multiplicity is implicit in how many links you actually draw.

## 🔹 Real-World Examples

### Customer / Order Snapshot

Scenario: Alice placed two orders, one shipped and one pending. Each order has line items.

```
┌─────────────────────┐
│ alice : Customer     │
├─────────────────────┤
│ name = "Alice"       │
│ loyaltyTier = "Gold" │
└────────┬──────┬─────┘
         │      │
         │      │
         v      v
┌──────────────────┐   ┌──────────────────┐
│ order42 : Order   │   │ order43 : Order   │
├──────────────────┤   ├──────────────────┤
│ date = 2026-06-01 │   │ date = 2026-07-04 │
│ status = "Shipped"│   │ status = "Pending"│
└───────┬──────────┘   └───────┬──────────┘
        │                      │
        v                      v
┌──────────────────┐   ┌──────────────────┐
│ : LineItem        │   │ : LineItem        │
├──────────────────┤   ├──────────────────┤
│ product = "Mouse" │   │ product = "Cable" │
│ qty = 2           │   │ qty = 1           │
│ price = 29.99     │   │ price = 12.50     │
└──────────────────┘   └──────────────────┘
```

Notice: the line items are **anonymous objects** (`: LineItem`) because their identity is not important for this snapshot.

### Department / Employee Instance Diagram

Scenario: Engineering department with two employees, one of whom manages the other.

```
┌─────────────────────────┐
│ eng : Department         │
├─────────────────────────┤
│ name = "Engineering"     │
│ budget = 500000          │
└──────┬──────────┬───────┘
       │          │
       v          v
┌──────────────┐  ┌──────────────┐
│ sam : Employee│  │ lee : Employee│
├──────────────┤  ├──────────────┤
│ name = "Sam"  │  │ name = "Lee"  │
│ role = "Lead" │  │ role = "Dev"  │
│ salary = 95000│  │ salary = 80000│
└──────┬───────┘  └──────────────┘
       │  manages        ^
       └─────────────────┘
```

The `manages` link is a self-association on Employee in the [[Class Diagram]], but here it connects two **specific** Employee instances.

## 🔹 When to Use Object Diagrams Over Class Diagrams

Object diagrams earn their keep in situations where the abstract class structure is not enough:

| Situation                              | Why an object diagram helps                                                  |
| -------------------------------------- | ---------------------------------------------------------------------------- |
| **Debugging a specific bug**           | Draw the exact object graph that caused the issue -- values and all          |
| **Explaining a scenario to the team**  | Concrete examples are easier to follow than abstract multiplicity rules      |
| **Validating multiplicity constraints**| "Can this 1..* really happen?" -- draw it and see if it makes sense          |
| **Illustrating design patterns**       | Show how Singleton, Composite, or Observer look at runtime with real objects |
| **Onboarding new developers**          | A snapshot of "here is what memory looks like after login" beats theory      |
| **Test case design**                   | Each object diagram maps naturally to one test scenario's setup              |

**Rule of thumb:** if someone asks "can you give me an example?", that is the moment to reach for an object diagram instead of pointing at the class diagram again.

See also: [[Common Notation]], [[Relationships]], [[Class Diagram]], [[Sequence Diagram]], [[Component Diagram]]
