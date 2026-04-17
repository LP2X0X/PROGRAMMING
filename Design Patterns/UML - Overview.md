---
tags: 
- design
- UML
---
# UML (Unified Modeling Language)

- **UML** is a standardized way to draw diagrams about software. It gives developers a common visual language to describe system structure, behavior, and interactions.
- You do not need to know all 14 diagram types. In practice, **5 diagrams** cover the vast majority of what you will encounter and need to draw.

---

## Class Diagram (most important)

- Shows the **static structure** of a system: classes, their attributes, methods, and the relationships between them.
- This is the diagram you will see most often, especially alongside [[Design Patterns - Detailed Notes|design patterns]].

### Class box

- A class is drawn as a box with three sections:

```
┌──────────────────┐
│    ClassName      │  ← name
├──────────────────┤
│ - privateField    │  ← attributes
│ + publicField     │
├──────────────────┤
│ + doSomething()   │  ← methods
│ - helper()        │
└──────────────────┘
```

### Visibility modifiers

| Symbol | Meaning          |
| ------ | ---------------- |
| `+`    | public           |
| `-`    | private          |
| `#`    | protected        | 
| `~`    | package/internal |

### Relationships

| Arrow        | Name        | Plain English                 | Example                    |
| ------------ | ----------- | ----------------------------- | -------------------------- |
| `A ——> B`    | Association | A knows about B               | `Customer ——> Order`       |
| `A ◆——> B`   | Composition | A **owns** B (B dies with A)  | `House ◆——> Room`          |
| `A ◇——> B`   | Aggregation | A **has** B (B can outlive A) | `Team ◇——> Player`         |
| `A ──▷ B`    | Inheritance | A **is a** B                  | `Dog ──▷ Animal`           |
| `A - - -▷ B` | Implements  | A implements interface B      | `ArrayList - - -▷ List`    |
| `A - - -> B` | Dependency  | A **uses** B temporarily      | `Controller - - -> Logger` |

```ad-tip
**How to remember Composition vs Aggregation:**
- **Filled diamond (◆) = Composition** — strong ownership. The room cannot exist without the house.
- **Empty diamond (◇) = Aggregation** — weak ownership. The player can exist without the team.
```

### Multiplicity

- Numbers placed on the ends of relationship lines to indicate how many objects participate:

| Notation      | Meaning        |
| ------------- | -------------- |
| `1`           | exactly one    |
| `0..1`        | zero or one    |
| `*` or `0..*` | zero or many   |
| `1..*`        | one or more    |
| `3..7`        | specific range |

- Example: `Customer 1 ◆——> * Order` means one customer owns many orders.

---

## Sequence Diagram (second most important)

- Shows **objects exchanging messages over time**. Time flows downward.
- Answers the question: **"what calls what, in what order?"**

```
 Customer        OrderService        Database
    │                 │                  │
    │── placeOrder() ─>│                  │
    │                 │── save() ────────>│
    │                 │<── confirmation ──│
    │<── receipt ─────│                  │
    │                 │                  │
```

### Key elements

| Element              | How it looks               | Meaning                                  |
| -------------------- | -------------------------- | ---------------------------------------- |
| Lifeline             | vertical dashed line       | the object exists during this time       | 
| Activation box       | thin rectangle on lifeline | the object is actively doing work        |
| Solid arrow          | `──>`                      | synchronous call (caller waits)          |
| Dashed arrow         | `- - ->`                   | return value                             |
| Open arrowhead       | `──>>`                     | asynchronous call (caller does not wait) |
| X at end of lifeline | `X`                        | object is destroyed                      |

### Fragments (combined fragments)

- Boxes around parts of the sequence to show control flow:
  - **alt** — if/else (alternative paths)
  - **opt** — optional (if without else)
  - **loop** — repetition
  - **par** — parallel execution

```ad-note
Sequence diagrams are excellent for documenting API call flows, debugging race conditions, and explaining how a feature works at runtime.
```

---

## Activity Diagram (fancy flowchart)

- Shows the **flow of control** from activity to activity. Essentially a flowchart with UML notation.
- **When to use:** modeling workflows, business processes, or algorithm logic.

### Symbols

| Symbol    | Shape                  | Meaning                                |
| --------- | ---------------------- | -------------------------------------- |
| Start     | filled black circle    | entry point                            |
| End       | black circle with ring | exit point                             |
| Action    | rounded rectangle      | a step or activity                     |
| Decision  | diamond                | if/else branch                         |
| Fork/Join | thick horizontal bar   | split into / merge from parallel paths |
| Swimlane  | vertical partition     | which actor or system is responsible   |

```
  (●)
   │
   ▼
┌──────────┐
│ Receive   │
│ Order     │
└────┬─────┘
     ▼
   ◆ In stock? ◆
  yes/        \no
   ▼           ▼
┌────────┐  ┌──────────┐
│ Ship   │  │ Back-    │
│ Item   │  │ order    │
└───┬────┘  └────┬─────┘
    └──────┬─────┘
           ▼
        (◉) end
```

---

## Use Case Diagram (the simplest one)

- Shows **what the system does for its users** at a high level. Does not show how.
- **When to use:** early requirements gathering, communicating scope to stakeholders.

### Elements

| Element         | Symbol                 | Meaning                              |
| --------------- | ---------------------- | ------------------------------------ |
| Actor           | stick figure           | a user or external system            |
| Use case        | oval                   | something the system does            |
| System boundary | rectangle around ovals | what is inside vs outside the system |
| Association     | line                   | actor participates in use case       | 
| `<<include>>`   | dashed arrow           | use case always includes another     |
| `<<extend>>`    | dashed arrow           | use case optionally extends another  |

```
            ┌───────────────────────┐
            │   Online Store        │
  👤 ───────│── (Place Order)       │
  Customer  │── (Track Order)       │
            │── (Cancel Order)      │
            │                       │
  👤 ───────│── (Manage Inventory)  │
  Admin     │── (Generate Reports)  │
            └───────────────────────┘
```

```ad-important
Use case diagrams show **what**, not **how**. They are not flowcharts. Each oval is a goal the actor wants to achieve, not a step in a process.
```

---

## State Diagram (state machine)

- Shows the **lifecycle of a single object** — what states it can be in and what triggers transitions between them.
- **When to use:** modeling objects with distinct states (orders, user sessions, network connections, UI components).

### Elements

| Element    | Symbol                   | Meaning                              |
| ---------- | ------------------------ | ------------------------------------ |
| State      | rounded rectangle        | a condition the object is in         |
| Transition | arrow with label         | trigger that causes state change     |
| Start      | filled black circle      | initial state                        |
| End        | black circle with ring   | final state                          | 
| Guard      | `[condition]` on arrow   | transition only if condition is true |
| Action     | `/ doSomething` on arrow | action performed during transition   |

### Example: Order lifecycle

```
 (●)
  │
  ▼
┌──────────┐   pay()    ┌──────────┐   ship()   ┌──────────┐
│ Pending  │ ──────────> │  Paid    │ ─────────> │ Shipped  │
└──────────┘             └──────────┘            └────┬─────┘
      │                       │                       │
      │ cancel()              │ cancel() / refund()   │ deliver()
      ▼                       ▼                       ▼
┌──────────┐             ┌──────────┐            ┌──────────┐
│Cancelled │             │ Refunded │            │Delivered │
└──────────┘             └──────────┘            └────┬─────┘
                                                      ▼
                                                    (◉)
```

---

## When to Use Which Diagram

| You want to show...               | Use this             |
| --------------------------------- | -------------------- |
| Code structure and relationships  | **Class diagram**    |
| Runtime call flow between objects | **Sequence diagram** |
| Workflow or process logic         | **Activity diagram** |
| What the system does for users    | **Use case diagram** |
| Lifecycle of one object           | **State diagram**    |

---

## The Other UML Diagrams (for reference)

- These are less commonly used day-to-day but exist in the spec:

| Diagram               | What it shows                                          |
| --------------------- | ------------------------------------------------------ |
| Component diagram     | High-level modules and their interfaces                |
| Deployment diagram    | Physical hardware and software mapping                 |
| Package diagram       | How code is organized into packages/namespaces         |
| Object diagram        | Snapshot of instances at a point in time               |
| Composite structure   | Internal structure of a class                          |
| Timing diagram        | State changes along a time axis                        |
| Communication diagram | Same as sequence but emphasizes links over time        | 
| Interaction overview  | Activity diagram where each node is a sequence diagram |
| Profile diagram       | Extends UML itself with stereotypes                    |

```ad-tip
You can go a long way knowing only Class, Sequence, and Activity diagrams. Add State and Use Case when needed. The rest are situational.
```
