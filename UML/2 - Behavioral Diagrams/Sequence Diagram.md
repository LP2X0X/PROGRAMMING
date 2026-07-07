---
tags:
  - uml
  - sequence-diagram
  - behavioral
---

# Sequence Diagram

A **sequence diagram** shows how objects interact through **messages** ordered in **time**. The vertical axis is time (top to bottom), and horizontal placement shows participants. It is the most widely used UML interaction diagram — you will encounter and draw these constantly in API design, protocol specification, debugging, code reviews, and architecture discussions.

> [!tip] If you only learn two UML diagrams, make them the **class diagram** and the **sequence diagram**. They cover structure and behavior respectively, and together they communicate most software designs.

---

## 🔹 When to Use

| Situation                                  | Sequence Diagram Useful? |
| ------------------------------------------ | :----------------------: |
| Designing API call flows                   |           Yes            |
| Documenting protocol exchanges (HTTP, TCP) |           Yes            |
| Debugging a multi-component issue          |           Yes            |
| Explaining how a feature works to the team |           Yes            |
| Modeling all possible states of an object  |            No            |
| Showing system structure / ownership       |            No            |

---

## 🔹 Lifelines

A **lifeline** represents a participant in the interaction — an object, a component, an actor, or a system.

```
  ┌──────────┐   ┌──────────┐   ┌──────────┐
  │  Client  │   │  Server  │   │ Database │
  └────┬─────┘   └────┬─────┘   └────┬─────┘
       │              │              │
       │              │              │          ← dashed vertical line
       │              │              │            = lifeline (passage of time)
       │              │              │
```

- The **box at the top** is the lifeline head — shows participant name and optionally type: `name : Type`
- The **dashed vertical line** below represents the participant's existence over time
- An **actor** lifeline uses the stick figure instead of a box

---

## 🔹 Messages

Messages are the horizontal arrows between lifelines. This is the core of the diagram.

### Message Types

```
  ┌────────┐                      ┌────────┐
  │ Client │                      │ Server │
  └───┬────┘                      └───┬────┘
      │                               │
      │────── doWork() ──────────────>│   Synchronous (filled arrowhead)
      │                               │   Sender BLOCKS until return
      │                               │
      │<─ ─ ─ result ─ ─ ─ ─ ─ ─ ─ ─│   Return (dashed line, open arrowhead)
      │                               │   Response to synchronous call
      │                               │
      │─ ─ ─ notify() ─ ─ ─ ─ ─ ─ ─>│   Asynchronous (open/stick arrowhead)
      │                               │   Sender does NOT block
      │                               │
```

| Arrow Style                     | Type         | Meaning                           |
| ------------------------------- | ------------ | --------------------------------- |
| `──────────────────────────>`    | Synchronous  | Caller blocks, waits for return   |
| `─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ >`   | Asynchronous | Fire-and-forget, caller continues |
| `<─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ `   | Return       | Response to a synchronous call    |
| `──────────────────────────>x`  | Lost         | Message sent but never received   |
| `x─────────────────────────>`   | Found        | Message received from unknown     |

> [!warning] Common Mistake
> Return messages are **dashed** lines with an **open** arrowhead. Synchronous messages are **solid** lines with a **filled** arrowhead. Mixing these up is the most frequent sequence diagram error.

### Create and Destroy Messages

```
  ┌────────┐                         
  │ Factory│                         
  └───┬────┘                         
      │                              
      │──── <<create>> ────────────> ┌──────────┐
      │                              │  Widget  │
      │                              └────┬─────┘
      │                                   │
      │                                   │
      │──── destroy() ──────────────────> X   ← destruction (large X)
      │                                   
```

- **Create**: arrow points to the lifeline head box (the box is drawn at the point of creation, not at the top)
- **Destroy**: large **X** on the lifeline where it ceases to exist

---

## 🔹 Activation Bars (Execution Specification)

A thin rectangle on the lifeline showing when the participant is **actively executing** (processing a message).

```
  ┌────────┐          ┌────────┐          ┌──────────┐
  │ Client │          │ Server │          │ Database │
  └───┬────┘          └───┬────┘          └────┬─────┘
      │                   │                    │
      │  request()        │                    │
      │──────────────────>│                    │
      │                   ┌┤                   │
      │                   ││  query()          │
      │                   ││──────────────────>│
      │                   ││                   ┌┤
      │                   ││                   ││
      │                   ││<─ ─ ─ rows ─ ─ ─ ┘│
      │                   ││                    │
      │<─ ─ ─ response ─ ┘│                    │
      │                    │                    │
```

- Activation bars stack for **nested calls** (one object calls another, which calls another)
- The bar starts when the message arrives and ends when the return fires (or processing completes)

---

## 🔹 Self-Messages

An object sending a message to itself — common for internal method calls.

```
  ┌────────┐
  │ Server │
  └───┬────┘
      │
      ┌┤
      ││──┐
      ││  │ validate()
      ││<─┘
      ││
      ┘│
      │
```

- Draw as a U-shaped arrow that leaves and returns to the same lifeline
- Creates a **nested activation bar** (thinner bar overlapping the existing one)
- Use for: internal validation, helper method calls, recursive operations

---

## 🔹 Combined Fragments

Combined fragments overlay a region of the sequence diagram with a labeled frame, adding control flow logic. This is where sequence diagrams become genuinely powerful.

### Fragment Notation

```
  ┌─────────────────────────────────────────────┐
  │ alt      │                                   │
  │──────────┘                                   │
  │ [condition1]                                 │
  │     ... messages ...                         │
  │─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
  │ [condition2]                                 │
  │     ... messages ...                         │
  │─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─│
  │ [else]                                       │
  │     ... messages ...                         │
  └─────────────────────────────────────────────┘
```

- The **operator** goes in the top-left pentagon
- **Guard conditions** in square brackets `[condition]`
- Dashed lines separate **operands** (branches)

### Fragment Types Reference

| Operator   | Meaning                      | Operands | Use When                                           |
| ---------- | ---------------------------- | :------: | -------------------------------------------------- |
| **alt**    | If/else (alternatives)       |   2+     | Different paths based on conditions                |
| **opt**    | Optional (if, no else)       |    1     | Something that may or may not happen               |
| **loop**   | Repeat                       |    1     | Iterating over a collection, retry logic           |
| **par**    | Parallel                     |   2+     | Concurrent operations                              |
| **critical** | Atomic / mutex             |    1     | Must execute without interleaving                  |
| **ref**    | Reference to another diagram |    1     | Reuse a sub-sequence defined elsewhere             |
| **break**  | Exit the enclosing fragment  |    1     | Error/exception — stop the current interaction     |
| **neg**    | Invalid / forbidden          |    1     | Documenting what must NOT happen                   |
| **assert** | Must happen exactly this way |    1     | The only valid continuation                        |
| **seq**    | Weak sequencing (default)    |   2+     | Ordering within each lifeline maintained            |
| **strict** | Strict sequencing            |   2+     | Total ordering across all lifelines                |
| **ignore** | Messages to ignore           |    1     | Abstracting away irrelevant messages               |
| **consider** | Messages to consider       |    1     | Only these messages matter                         |

> [!tip] In practice, you'll use **alt**, **opt**, **loop**, **ref**, and **break** for 95% of your diagrams. The rest are niche.

---

### alt (Alternatives — if/else)

```
  ┌────────┐          ┌────────┐
  │ Client │          │ Server │
  └───┬────┘          └───┬────┘
      │                   │
      │  login(user,pass) │
      │──────────────────>│
      │                   │
      │ ┌─────────────────┤─── alt ──┐
      │ │ [valid]         │          │
      │ │<── token ──── ─ │          │
      │ ├─ ─ ─ ─ ─ ─ ─ ─ ┤ ─ ─ ─ ─ ┤
      │ │ [invalid]       │          │
      │ │<── 401 ──── ─ ─ │          │
      │ └─────────────────┤──────────┘
      │                   │
```

### opt (Optional — if, no else)

```
      │ ┌─────────────────┤─── opt ──┐
      │ │ [hasDiscount]   │          │
      │ │  applyDiscount()│          │
      │ │────────────────>│          │
      │ └─────────────────┤──────────┘
```

### loop (Repeat)

```
      │ ┌─────────────────┤── loop ──┐
      │ │ [for each item]  │          │
      │ │  process(item)   │          │
      │ │─────────────────>│          │
      │ │<─ ─ ─ ok ─ ─ ─ ─│          │
      │ └─────────────────┤──────────┘
```

- You can specify bounds: `loop(1, 10)` = min 1, max 10 iterations
- `loop(0, *)` = zero or more (the default if no bounds given)

### par (Parallel)

```
      │ ┌──────────────────────────────┤── par ──┐
      │ │  fetchUserProfile()          │          │
      │ │─────────────────────────────>│          │
      │ ├─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─┤ ─ ─ ─ ─ ┤
      │ │  fetchUserOrders()           │          │
      │ │─────────────────────────────>│          │
      │ └──────────────────────────────┤──────────┘
```

- Both regions execute **concurrently** — no guaranteed order
- Used for: parallel API calls, concurrent processing, async operations

### ref (Reference)

```
      │ ┌─────────────────────────────── ref ────┐
      │ │                                        │
      │ │        Authenticate User               │
      │ │                                        │
      │ └────────────────────────────────────────┘
```

- Points to another sequence diagram defined elsewhere
- Excellent for **reuse** — define "Authenticate User" once, reference it in many diagrams
- Keeps diagrams manageable by extracting common sub-sequences

### break (Exit)

```
      │ ┌─────────────────┤── break ─┐
      │ │ [error]         │          │
      │ │<── 500 ──── ─ ─ │          │
      │ └─────────────────┤──────────┘
      │                   │
      │   (remaining      │
      │    messages here  │
      │    are SKIPPED)   │
```

- If the guard is true, the messages inside **break** execute and then the **enclosing fragment is exited** — remaining messages after break are skipped
- Think of it like a `break` statement or `throw` — it short-circuits

### neg (Negative / Invalid)

```
      │ ┌─────────────────┤── neg ───┐
      │ │  directDBAccess()│          │
      │ │─────────────────>│          │
      │ └─────────────────┤──────────┘
```

- Documents interactions that **must NOT occur** — useful in security specs
- This is a constraint, not an execution path

---

## 🔹 Example: HTTP Request/Response

```
  ┌────────┐          ┌────────┐          ┌──────────┐
  │ Browser│          │ Server │          │ Database │
  └───┬────┘          └───┬────┘          └────┬─────┘
      │                   │                    │
      │ GET /users        │                    │
      │──────────────────>│                    │
      │                   ┌┤                   │
      │                   ││ SELECT * FROM     │
      │                   ││ users             │
      │                   ││──────────────────>│
      │                   ││                   ┌┤
      │                   ││                   ││
      │                   ││<─ ─ ─ rows ─ ─ ─ ┘│
      │                   ││                   │
      │                   ││──┐                │
      │                   ││  │ serialize      │
      │                   ││  │ ToJSON()       │
      │                   ││<─┘                │
      │                   ││                   │
      │<─ ─ 200 JSON ─ ─ ┘│                   │
      │                    │                   │
```

---

## 🔹 Example: Login Flow with Error Handling

```
  ┌──────┐        ┌──────────┐      ┌──────────┐     ┌──────────┐
  │ User │        │   App    │      │ AuthSvc  │     │    DB    │
  └──┬───┘        └────┬─────┘      └────┬─────┘     └────┬─────┘
     │                 │                  │                │
     │ login(u, p)     │                  │                │
     │────────────────>│                  │                │
     │                 │  authenticate()  │                │
     │                 │─────────────────>│                │
     │                 │                  │  findUser(u)   │
     │                 │                  │───────────────>│
     │                 │                  │<─ ─ user ─ ─ ─│
     │                 │                  │                │
     │                 │ ┌────────────────┤── alt ────────┐│
     │                 │ │ [user found    │               ││
     │                 │ │  && pass match]│               ││
     │                 │ │                │               ││
     │                 │ │  createToken() │               ││
     │                 │ │<─ ─ token ─ ─ ─│               ││
     │                 │ │                │               ││
     │<─ ─ 200 + token─│ │               │               ││
     │                 │ ├─ ─ ─ ─ ─ ─ ─ ─┤ ─ ─ ─ ─ ─ ─ ─┤│
     │                 │ │ [else]         │               ││
     │                 │ │                │               ││
     │                 │ │  logAttempt()  │               ││
     │                 │ │───────────────>│               ││
     │                 │ │                │               ││
     │<─ ─ 401 ─ ─ ─ ─│ │               │               ││
     │                 │ └────────────────┤───────────────┘│
     │                 │                  │                │
```

---

## 🔹 Example: E-Commerce Checkout

```
  ┌──────┐     ┌──────────┐    ┌───────────┐   ┌─────────┐   ┌──────────┐
  │ User │     │   Cart   │    │ OrderSvc  │   │ PayGate │   │ EmailSvc │
  └──┬───┘     └────┬─────┘    └─────┬─────┘   └────┬────┘   └────┬─────┘
     │              │                │               │             │
     │ checkout()   │                │               │             │
     │─────────────>│                │               │             │
     │              │ createOrder()  │               │             │
     │              │───────────────>│               │             │
     │              │                │               │             │
     │              │ ┌──────────────┤─── loop ─────┐│             │
     │              │ │[each item]   │              ││             │
     │              │ │              │              ││             │
     │              │ │──┐           │              ││             │
     │              │ │  │validateQty│              ││             │
     │              │ │<─┘           │              ││             │
     │              │ │              │              ││             │
     │              │ │ ┌────────────┤── break ────┐││             │
     │              │ │ │[outOfStock]│             │││             │
     │              │ │ │            │             │││             │
     │<─ ─ error ─ ─ ─ │            │             │││             │
     │              │ │ └────────────┤─────────────┘││             │
     │              │ └──────────────┤──────────────┘│             │
     │              │                │               │             │
     │              │                │ charge()      │             │
     │              │                │──────────────>│             │
     │              │                │<─ ─ receipt ─ │             │
     │              │                │               │             │
     │              │                │ ┌─────────────┤── opt ─────┐│
     │              │                │ │[email opted] │            ││
     │              │                │ │ sendConfirm()│            ││
     │              │                │ │─────────────────────────>││
     │              │                │ └─────────────┤────────────┘│
     │              │                │               │             │
     │<─ ─ ─ ─ confirmation ─ ─ ─ ─ │               │             │
     │              │                │               │             │
```

---

## 🔹 Sequence Diagram vs Communication Diagram

Both show the same information — object interactions — but emphasize different aspects.

| Aspect               | Sequence Diagram                         | [[Communication Diagram]]                  |
| --------------------- | ---------------------------------------- | ------------------------------------------ |
| **Emphasis**          | Time ordering of messages                | Structural relationships between objects   |
| **Layout**            | Vertical (time flows down)               | Free-form (objects placed spatially)       |
| **Message ordering**  | Implicit via vertical position           | Explicit via numbering scheme              |
| **Complex control**   | Combined fragments (alt, loop, etc.)     | Limited — guards and iteration only        |
| **Scalability**       | Gets wide with many participants         | Gets cluttered with many messages          |
| **Best for**          | Detailed interactions, protocol specs    | Showing who talks to whom (overview)       |
| **Tool support**      | Excellent — most tools support it        | Decent but less popular                    |
| **When to choose**    | When order/timing matters most           | When connections/topology matters most     |

> [!tip] ==Start with a sequence diagram.== If you find the temporal ordering is not the point and you really want to show *which objects are connected*, switch to a communication diagram.

---

## 🔹 Numbering and Naming Conventions

- **Lifeline names**: `objectName : ClassName` — e.g., `cart : ShoppingCart`. You can omit either the name or the type.
- **Message names**: use method-call syntax for synchronous calls — `calculate(amount, tax)`. Use signal names for async — `orderPlaced`.
- **Return values**: label the return arrow with the value — `result`, `true`, `List<User>`.

---

## 🔹 Practical Tips

> [!tip] Keep it readable
> Limit to **5-7 lifelines** per diagram. More than that becomes unreadable. Use `ref` fragments to extract sub-sequences into separate diagrams.

> [!tip] Show the happy path first
> Draw the main success scenario, then add `alt` / `break` fragments for error handling. Don't try to show every edge case in one diagram.

> [!tip] Use color and notes
> Most tools let you add notes (sticky notes attached to messages) and color-code lifelines. Use them — a note saying "this call takes 200ms on average" is invaluable.

> [!warning] Common Mistake
> Don't confuse the *return arrow* with a new message going the other direction. A dashed arrow labeled with a return value is not a separate call — it's the response to the synchronous message above it.

> [!warning] Common Mistake
> Forgetting activation bars makes it unclear when processing starts and stops. Always include them for synchronous calls.

---

See also: [[Communication Diagram]], [[Activity Diagram]], [[State Machine Diagram]], [[Interaction Overview Diagram]], [[Timing Diagram]]
