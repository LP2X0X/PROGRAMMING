---
tags:
  - uml
  - communication-diagram
  - behavioral
---

## 🔹 What It Shows

A communication diagram shows the same information as a [[Sequence Diagram]] -- object interactions via messages -- but emphasizes the **structural links between objects** rather than time ordering. Instead of vertical lifelines, you see objects connected by lines (links), with numbered messages written along those links.

> Previously called a **Collaboration Diagram** in UML 1.x. Renamed to Communication Diagram in UML 2.0.

Because messages are numbered instead of laid out top-to-bottom, communication diagrams make it easy to see **which objects talk to each other** at a glance, but harder to follow the exact chronological flow.

## 🔹 Core Elements

| Element | Notation | Purpose |
|---------|----------|---------|
| **Object** | `objectName : ClassName` in a box | An instance participating in the interaction |
| **Link** | Solid line between objects | A connection (usually an [[Relationships\|association]]) that messages travel along |
| **Message** | Arrow + sequence number + name | A call or signal sent along a link |
| **Return** | Dashed arrow (often omitted) | Value coming back from a message |
| **Actor** | Stick figure | External entity initiating the interaction |

Objects use the same underlined-instance notation from [[Common Notation]]. Anonymous objects omit the name: `: ClassName`.

## 🔹 Objects, Links, and Messages

The fundamental building block is: two objects connected by a link, with numbered messages flowing along it.

```
┌──────────┐    1: login(uid, pwd)    ┌───────────┐
│  :Client  │ ───────────────────────> │  :AuthSvc  │
└──────────┘                          └───────────┘
```

Multiple messages can travel along the **same link** in either direction -- just stack the numbered arrows:

```
┌──────────┐  1: login(uid, pwd)   ┌───────────┐
│  :Client  │ ──────────────────── > │  :AuthSvc  │
│           │ < ────────────────── │            │
└──────────┘  2: token             └───────────┘
```

## 🔹 Message Numbering Schemes

Numbering is what gives a communication diagram its sense of order. Two styles exist:

### Flat Numbering

Messages are numbered sequentially: `1, 2, 3, 4 ...`

Simple, but you lose visibility into which message **triggered** which.

```
┌──────────┐  1: placeOrder()  ┌───────────┐  2: charge()  ┌──────────┐
│  :Client  │ ───────────────> │  :OrderSvc │ ────────────> │ :Payment  │
└──────────┘                   └───────────┘                └──────────┘
                                     │
                                     │ 3: ship()
                                     v
                               ┌───────────┐
                               │ :Shipping  │
                               └───────────┘
```

### Nested / Decimal Numbering (More Common)

Messages use dotted notation: `1.1, 1.2, 1.1.1 ...`

The nesting shows **call hierarchy** -- which message was sent as part of handling which other message.

| Number | Meaning |
|--------|---------|
| `1` | First top-level message |
| `1.1` | First message sent while handling message 1 |
| `1.2` | Second message sent while handling message 1 |
| `1.1.1` | First message sent while handling message 1.1 |
| `2` | Second top-level message (after 1 fully completes) |

```
┌──────────┐  1: placeOrder()    ┌───────────┐  1.1: charge()  ┌──────────┐
│  :Client  │ ─────────────────> │  :OrderSvc │ ──────────────> │ :Payment  │
└──────────┘                     └───────────┘                  └──────────┘
                                       │
                                       │ 1.2: ship()
                                       v
                                 ┌───────────┐
                                 │ :Shipping  │
                                 └───────────┘
```

Here `charge()` and `ship()` are both triggered **inside** `placeOrder()`, and `charge()` happens before `ship()`.

## 🔹 Guards and Iteration

### Guards

A **guard** is a boolean condition in square brackets. The message is only sent if the condition is true.

```
1.1 [isValid]: save()
1.2 [!isValid]: reject()
```

### Iteration

Prefix a message with `*` and an optional iteration clause to show a loop.

```
1.1 *[for each item]: validate(item)
```

Combined example along a link:

```
┌───────────┐  1.1 *[for each item]: addLine(item)  ┌──────────┐
│  :OrderSvc │ ────────────────────────────────────> │  :Order   │
└───────────┘                                        └──────────┘
```

## 🔹 Self-Delegation

An object can send a message to itself. Draw a small loop arrow back to the same object:

```
             1.1: validate()
           ┌──────┐
           │      │
           v      │
      ┌───────────┐
      │   :Order   │
      └───────────┘
```

This is the communication-diagram equivalent of a self-call on a [[Sequence Diagram]] lifeline. Use it when an object's method internally calls another of its own methods.

## 🔹 Real-World Example: Order Processing

A customer places an order. The system validates inventory, charges payment, and sends a confirmation email.

```
                        1: placeOrder(cart)
  ┌──────────┐  ─────────────────────────────────>  ┌─────────────┐
  │ :Customer │                                      │  :OrderSvc   │
  └──────────┘  <─────────────────────────────────  └─────────────┘
                     1.4: confirmation(orderNum)        │       │
                                                        │       │
                                          1.1: check(items)     │
                                                │       │       │
                                                v       │       │
                                         ┌────────────┐ │       │
                                         │ :Inventory  │ │       │
                                         └────────────┘ │       │
                                                        │       │
                                                        │  1.2 [inStock]: charge(total)
                                                        │       │
                                                        │       v
                                                        │  ┌──────────┐
                                                        │  │ :Payment  │
                                                        │  └──────────┘
                                                        │
                                                   1.3: sendConfirmation(order)
                                                        │
                                                        v
                                                  ┌───────────┐
                                                  │ :EmailSvc  │
                                                  └───────────┘
```

**Reading the numbers (nested scheme):**

| # | Message | Meaning |
|---|---------|---------|
| 1 | `placeOrder(cart)` | Customer initiates the order |
| 1.1 | `check(items)` | OrderSvc asks Inventory to verify stock |
| 1.2 | `[inStock]: charge(total)` | If in stock, OrderSvc charges via Payment |
| 1.3 | `sendConfirmation(order)` | OrderSvc tells EmailSvc to email the customer |
| 1.4 | `confirmation(orderNum)` | OrderSvc returns the order number to Customer |

Notice how the **links** make it obvious that OrderSvc is the central coordinator -- something that's harder to spot on a [[Sequence Diagram]] where it's just one lifeline among many.

## 🔹 Communication vs Sequence: When to Use Which

| Criterion | Communication Diagram | [[Sequence Diagram]] |
|-----------|----------------------|----------------------|
| **Best for showing** | Object relationships / who talks to whom | Exact time-ordered flow of messages |
| **Layout** | Free-form (2D network) | Structured top-to-bottom timeline |
| **Scales well when** | Few objects, many links | Many messages in a linear flow |
| **Struggles when** | Many sequential messages (numbering gets dense) | Many objects (lifelines crowd horizontally) |
| **Call hierarchy** | Visible via decimal numbering (1.1, 1.2) | Visible via nesting depth of activation bars |
| **Concurrency** | Hard to express | Easy with parallel fragments (`par`) |
| **Fragments (alt/loop/opt)** | Not supported -- use guards/iteration instead | Full combined-fragment support |
| **Typical audience** | Architects reviewing object coupling | Developers implementing step-by-step logic |
| **Quick rule of thumb** | "Who connects to whom?" | "What happens in what order?" |

**Choose Communication when:**
- You want to highlight the **network of collaborating objects**
- The interaction is short and the object relationships matter more than timing
- You're reviewing whether objects have appropriate [[Relationships|associations]]

**Choose Sequence when:**
- The flow has many steps and order matters
- You need combined fragments (alt, loop, opt, par)
- You're detailing a specific use case or algorithm step by step

Both diagrams are **semantically equivalent** -- any communication diagram can be redrawn as a sequence diagram and vice versa. Pick whichever makes the point clearer for your audience.

See also: [[Common Notation]], [[Relationships]], [[Sequence Diagram]], [[Interaction Overview Diagram]], [[Activity Diagram]]
