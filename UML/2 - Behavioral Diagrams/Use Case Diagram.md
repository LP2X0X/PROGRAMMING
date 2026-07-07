---
tags:
  - uml
  - use-case-diagram
  - behavioral
---

# Use Case Diagram

A **use case diagram** shows what a system does from the *user's perspective* — not how it does it. It captures **functional requirements** as named goals (use cases) that **actors** can achieve through the system. Think of it as the "menu" of what your system offers to the outside world.

> [!tip] This is the most non-technical UML diagram — perfect for conversations with stakeholders who don't read code.

---

## 🔹 When to Use

| Situation                            | Use Case Diagram Useful? |
| ------------------------------------ | :----------------------: |
| Requirements gathering / elicitation |           Yes            |
| Communicating scope to stakeholders  |           Yes            |
| Defining system boundary             |           Yes            |
| Detailed algorithm design            |           No             |
| Object interaction / messaging       |           No             |
| UI mockups                           |           No             |

Use case diagrams answer: **"What can each type of user do with this system?"**

---

## 🔹 Core Elements

### Actor

An **actor** is anything external that interacts with the system — a person, another system, a hardware device, or a timer/scheduler.

```
     O        <<system>>
    /|\      ┌──────────┐
    / \      │ PayPal   │
  Customer   └──────────┘
 (primary)    (secondary)
```

- **Primary actor** — initiates the interaction (left side by convention)
- **Secondary/supporting actor** — the system calls upon it (right side)
- Actors are *roles*, not specific people. "Manager" and "Employee" are two actors even if the same person fills both.

### Use Case

A named oval representing a goal the actor can accomplish.

```
  ┌─────────────────────────┐
  │      (  Place Order  )  │
  │                         │
  │      (  View Cart  )    │
  └─────────────────────────┘
```

- Name with a **verb phrase**: "Place Order", not "Order" or "OrderManager"
- Each use case should deliver **observable value** to an actor

### System Boundary

A rectangle enclosing all use cases — everything inside is the system's responsibility.

```
  ┌──────────────────────────────┐
  │       Online Store           │
  │                              │
  │    (  Browse Catalog  )      │
  │    (  Place Order     )      │
  │    (  Track Shipment  )      │
  │                              │
  └──────────────────────────────┘
```

---

## 🔹 Relationships

### Association

A solid line between an actor and a use case — "this actor participates in this use case."

```
  Customer ──── ( Place Order )
```

- No arrowhead needed (both sides know about each other)
- An arrowhead is sometimes drawn pointing toward the use case to show who initiates, but it's optional

### Include (`<<include>>`)

Use case A **always** needs use case B as a mandatory substep.

```
  ( Place Order ) ─ ─ ─<<include>>─ ─ ─> ( Authenticate User )
```

- ==Arrow points FROM the base use case TO the included use case==
- The included use case **always executes** when the base runs
- Think of it as a **function call** — the base cannot complete without it

### Extend (`<<extend>>`)

Use case B **optionally** adds behavior to use case A under certain conditions.

```
  ( Place Order ) <─ ─ ─<<extend>>─ ─ ─ ( Apply Coupon )
```

- ==Arrow points FROM the extending use case TO the base use case== (opposite direction to include!)
- The extending behavior is **conditional** — it may or may not happen
- Think of it as a **plugin** — the base use case works fine without it

### Generalization (Inheritance)

One actor or use case is a specialized version of another.

```
     O                O
    /|\              /|\
    / \              / \
  Customer ───▷  User (general)
              △
```

- Hollow triangle arrowhead points to the **general** (parent)
- A "Gold Customer" generalizes "Customer" — it inherits all associations and adds its own

---

## 🔹 Include vs Extend — The Key Distinction

This is the single most confused topic in use case diagrams.

| Aspect             | `<<include>>`                            | `<<extend>>`                               |
| ------------------ | ---------------------------------------- | ------------------------------------------ |
| **Execution**      | Always runs                              | Conditionally runs                         |
| **Direction**      | Base ─ ─ ─> Included                     | Extending ─ ─ ─> Base                      |
| **Analogy**        | Function call                            | Plugin / hook                              |
| **Base depends?**  | Yes — base cannot work without it        | No — base works fine alone                 |
| **Who knows whom** | Base knows about the included            | Base does NOT know about the extension     |
| **Purpose**        | Factor out common mandatory behavior     | Add optional/conditional behavior          |
| **Example**        | "Place Order" includes "Process Payment" | "Place Order" extended by "Apply Coupon"   |

> [!warning] Common Mistake
> If you find yourself drawing `<<extend>>` for something that *always* happens, you want `<<include>>`. Extend is for truly optional behavior that the base use case doesn't depend on.

**Decision rule**: Ask yourself — *"Does the base use case still make sense if this other use case doesn't execute?"*
- **Yes** → `<<extend>>`
- **No** → `<<include>>`

---

## 🔹 Full Example: Online Shopping System

```
                            ┌──────────────────────────────────────────────┐
                            │           Online Shopping System             │
                            │                                              │
     O                      │   ( Browse Catalog )                         │
    /|\   ──────────────────│── ( Place Order )                            │
    / \                     │       │               ╲                      │
  Customer                  │       │ <<include>>    ╲ <<include>>         │
                            │       ▼                 ▼                    │
     O                      │   ( Authenticate )    ( Process Payment ) ──│──  <<system>>
    /|\   ──────────────────│── ( Manage Inventory )                      │   ┌──────────┐
    / \                     │                                              │   │ PayPal   │
   Admin                    │   ( Apply Coupon ) ─ ─<<extend>>─ ─ ─ ─ ─ ─>│   └──────────┘
                            │       ▲                                      │
                            │       │ (to Place Order)                     │
                            │                                              │
                            │   ( Track Shipment )                         │
                            │                                              │
                            └──────────────────────────────────────────────┘
```

Reading this diagram:
- **Customer** can Browse Catalog, Place Order, and Track Shipment
- **Admin** can Manage Inventory
- **Place Order** always includes Authenticate and Process Payment
- **Apply Coupon** optionally extends Place Order
- **PayPal** is a secondary actor that participates in Process Payment

---

## 🔹 Example: Library Management System

```
  ┌──────────────────────────────────────────────┐
  │          Library Management System            │
  │                                               │
  │     ( Search Book )                           │
  │     ( Borrow Book ) ──<<include>>──> ( Check  │
  │     ( Return Book )                  Member   │
  │     ( Reserve Book )                 Status ) │
  │                                               │
  │     ( Send Overdue Notice )                   │
  │     ( Generate Reports )                      │
  │                                               │
  │     ( Pay Fine ) ─ ─<<extend>>─ ─> ( Return   │
  │                                     Book )    │
  └──────────────────────────────────────────────┘

     O                        O               <<system>>
    /|\  ─── associations    /|\             ┌──────────┐
    / \      to top 4       / \              │  Email   │
  Member                  Librarian          │  Server  │
                          (associations      └──────────┘
                           to all)           (Send Overdue
                                              Notice)
```

---

## 🔹 Common Mistakes

> [!danger] Mistake 1: Wrong Granularity
> Don't model individual UI steps as use cases. "Click Login Button", "Enter Password", "Press Submit" are **not** use cases. The use case is "Log In" — it's a *goal*, not a step.

> [!danger] Mistake 2: Confusing Include/Extend Arrow Direction
> `<<include>>` arrow goes **from base to included**. `<<extend>>` arrow goes **from extension to base**. Reversing these completely changes the meaning.

> [!danger] Mistake 3: Treating Actors as Components
> "Database" is not an actor — it's inside your system. Actors are *external* to the system boundary.

> [!danger] Mistake 4: Too Many Use Cases
> A use case diagram with 50+ ovals is unreadable. Group related functionality or split into multiple diagrams by subsystem. Aim for 5-15 use cases per diagram.

> [!danger] Mistake 5: No System Boundary
> Without the rectangle, you can't tell what's inside vs outside the system. Always draw it.

---

## 🔹 Best Practices

- **Name use cases as verb phrases** — "Withdraw Cash", not "ATM Transaction"
- **One diagram per subsystem** if the system is large
- **Actors on the outside**, use cases on the inside — always
- **Primary actors on the left**, secondary on the right (convention)
- **Don't over-model** — use case diagrams are for communication, not exhaustive specification
- **Supplement with use case descriptions** — the diagram is the overview; write textual descriptions for the details (preconditions, postconditions, main flow, alternate flows)

---

See also: [[Activity Diagram]], [[Sequence Diagram]], [[State Machine Diagram]]
