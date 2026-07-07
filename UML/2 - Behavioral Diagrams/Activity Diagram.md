---
tags:
  - uml
  - activity-diagram
  - behavioral
---

## 🔹 What It Shows

An activity diagram models the **flow of control from activity to activity**. It captures the dynamic behavior of a system by showing the sequence of actions, decisions, and parallel paths that make up a workflow or business process.

- Visualizes **workflow and business process flow** -- how work moves through a system
- Shows the ordering of activities, including conditional branching and concurrent execution
- Represents both computational procedures (algorithms) and organizational processes (human workflows)
- Closest UML diagram to a traditional flowchart, but significantly more expressive

Unlike a [[Sequence Diagram]] which focuses on message ordering between objects, an activity diagram focuses on **what happens** and **in what order**, regardless of which object does it.

## 🔹 When to Use

| Scenario                          | Why Activity Diagrams Fit                              |
| --------------------------------- | ------------------------------------------------------ |
| Business process modeling         | Swimlanes show who does what                           |
| Algorithm logic                   | Decision nodes and loops map naturally                  |
| Use case flow elaboration         | Detail the steps inside a [[Use Case Diagram]] use case |
| Parallel/concurrent behavior      | Fork/join bars model true concurrency                  |
| Complex conditional logic         | Guard conditions on edges make branching explicit       |
| Data-driven workflows             | Object nodes show data moving between actions          |
| Exception and alternate flows     | Multiple paths with flow finals handle partial exits   |

## 🔹 Difference from Flowcharts

Activity diagrams look like flowcharts but are a proper UML construct with additional capabilities:

| Feature                | Flowchart        | Activity Diagram              |
| ---------------------- | ---------------- | ----------------------------- |
| Swimlanes/partitions   | Not standard     | Built-in for responsibilities |
| True concurrency       | No               | Fork/join bars                |
| Signals                | No               | Send/receive signal nodes     |
| Object flow            | No               | Object nodes on edges         |
| Standardized notation  | Varies           | UML standard                  |
| Flow final vs activity final | No distinction | Distinct nodes            |
| Pins (typed I/O)       | No               | Input/output pins on actions  |
| Interruptible regions  | No               | Supported                     |

A flowchart shows "step A then step B." An activity diagram can show "step A, then fork into B and C running simultaneously, with data object D flowing from B to E, all within the Customer partition."

## 🔹 Quick Reference: All Symbols

| Symbol             | Notation              | Meaning                                          |
| ------------------ | --------------------- | ------------------------------------------------ |
| Initial node       | Filled circle         | Entry point -- where the flow starts             |
| Activity final     | Filled circle in ring | Terminates the entire activity                   |
| Flow final         | Circle with X         | Ends one flow path without killing the activity  |
| Action             | Rounded rectangle     | A single atomic step of behavior                 |
| Decision node      | Diamond (1 in, N out) | Branch based on guard conditions                 |
| Merge node         | Diamond (N in, 1 out) | Rejoin multiple paths into one                   |
| Fork bar           | Thick bar (1 in, N out) | Split into concurrent parallel flows           |
| Join bar           | Thick bar (N in, 1 out) | Synchronize parallel flows back together       |
| Swimlane           | Vertical/horiz. lane  | Partition showing actor/role responsibility      |
| Object node        | Rectangle             | Data object flowing through the activity         |
| Send signal        | Convex pentagon       | Emit an asynchronous signal                      |
| Receive signal     | Concave pentagon      | Wait for an asynchronous signal                  |
| Pin                | Small square on edge  | Typed input/output port on an action             |
| Edge/flow          | Arrow                 | Direction of control or object flow              |
| Guard condition    | `[condition]` on edge | Boolean condition that must be true to traverse  |

## 🔹 Elements in Detail

### Initial Node

The starting point of the activity. Every activity diagram has exactly one initial node.

```
          ●
          |
          v
   +--------------+
   | First Action  |
   +--------------+
```

### Activity Final Node

Terminates the **entire** activity. When any flow reaches this node, all flows stop.

```
   +--------------+
   | Last Action   |
   +--------------+
          |
          v
         (●)       <-- filled circle inside a ring
```

### Flow Final Node

Ends **one specific path** without terminating the whole activity. Other concurrent paths continue.

```
   +--------------+
   |  Sub-Action   |
   +--------------+
          |
          v
         (X)        <-- circle with X, this path ends here
                        but other parallel paths keep going
```

### Action / Activity Node

A single unit of behavior. Named with a **verb phrase**. Drawn as a rounded rectangle.

```
   +-------------------+       +---------------------+
   |  Validate Input   |  -->  |  Process Payment    |
   +-------------------+       +---------------------+
```

### Decision Node

A diamond with **one incoming** edge and **multiple outgoing** edges. Each outgoing edge has a **guard condition** in square brackets. Guards must be mutually exclusive and collectively exhaustive.

```
                  |
                  v
                /   \
               / amt  \
              <  > 100? >
               \     /
                \   /
         [yes]  / \   [no]
               v       v
   +-----------+   +-----------+
   | Apply     |   | Standard  |
   | Discount  |   | Price     |
   +-----------+   +-----------+
```

### Merge Node

A diamond with **multiple incoming** edges and **one outgoing** edge. Reunites paths that were split by a decision. No guard conditions on the outgoing edge.

```
   +-----------+       +-----------+
   | Path A    |       | Path B    |
   +-----------+       +-----------+
         \                 /
          v               v
            \           /
              \       /
                \ _ /
                 | |
                  v
           +-----------+
           | Continue  |
           +-----------+
```

### Fork Bar

A thick horizontal bar with **one incoming** flow and **multiple outgoing** flows. Starts parallel concurrent execution.

```
           |
           v
   ================         <-- thick bar (fork)
      |         |
      v         v
  +-------+  +--------+
  | Task A |  | Task B |
  +-------+  +--------+
```

All outgoing flows run **simultaneously**. This is true concurrency, not a choice.

### Join Bar

A thick horizontal bar with **multiple incoming** flows and **one outgoing** flow. Waits for **all** incoming flows to complete before proceeding.

```
  +-------+  +--------+
  | Task A |  | Task B |
  +-------+  +--------+
      |         |
      v         v
   ================         <-- thick bar (join)
           |
           v
     +-----------+
     | Next Step |
     +-----------+
```

The flow after the join bar only fires when every incoming flow has arrived.

### Swimlanes / Partitions

Vertical or horizontal lanes that assign responsibility to actors, departments, or systems.

```
  |   Customer    |     System      |   Warehouse    |
  |               |                 |                |
  |     ●         |                 |                |
  |     |         |                 |                |
  |     v         |                 |                |
  | +---------+   |                 |                |
  | | Place   |   |                 |                |
  | | Order   |---+-->+---------+   |                |
  | +---------+   |   | Validate|   |                |
  |               |   | Order   |   |                |
  |               |   +---------+   |                |
  |               |       |         |                |
  |               |       v         |                |
  |               |   +---------+   |                |
  |               |   | Charge  |---+-->+---------+  |
  |               |   | Payment |   |   | Pack    |  |
  |               |   +---------+   |   | Items   |  |
  |               |                 |   +---------+  |
```

Swimlanes clarify **who** is responsible for each action. They can be vertical (as above) or horizontal.

### Object Nodes

Rectangles representing data objects that flow between actions. Shown on edges or as standalone nodes.

```
   +--------------+        +--------------+
   | Create Order |------->| Validate     |
   +--------------+        | Order        |
          |                +--------------+
          v
   +--------------+
   | [Order]      |    <-- object node (data flowing)
   | {created}    |        with state in curly braces
   +--------------+
          |
          v
   +--------------+
   | Process Order|
   +--------------+
```

### Signals: Send and Receive

**Send signal** -- a convex pentagon (arrow shape). Emits an asynchronous signal.
**Receive signal** -- a concave pentagon (notched shape). Waits for an incoming signal.

```
   Send Signal:                Receive Signal:

   +----------\                /----------+
   |  Send     \              /   Receive  |
   |  Invoice   >            <   Payment   |
   |           /              \            |
   +----------/                \----------+
```

Signals are useful for modeling asynchronous communication between independent processes.

### Pins

Small squares attached to the border of an action node. They represent typed input/output parameters.

```
                    [pin]              [pin]
                      |                  |
                      v                  v
              +---[=]----------[=]---+
              |                      |
              |   Calculate Total    |
              |                      |
              +---[=]----------------+
                    ^
                    |
                  [pin]

   [=] = small filled square on the action border

   Input pins:  data flowing INTO the action
   Output pins: data flowing OUT of the action
```

```
   Example with labeled pins:

       [items]         [taxRate]
          |                |
          v                v
   +---[=]------[=]---[=]---+
   |                         |
   |    Calculate Total      |
   |                         |
   +------[=]----------------+
            |
            v
        [totalAmt]
```

## 🔹 Control Flow Patterns

### 1. Sequential Flow

The simplest pattern -- actions execute one after another.

```
       ●
       |
       v
  +-----------+
  | Receive   |
  | Request   |
  +-----------+
       |
       v
  +-----------+
  | Process   |
  | Request   |
  +-----------+
       |
       v
  +-----------+
  | Send      |
  | Response  |
  +-----------+
       |
       v
      (●)
```

### 2. Decision / Branching with Guard Conditions

Use a decision diamond to branch and a merge diamond to rejoin.

```
          ●
          |
          v
   +--------------+
   | Check Stock  |
   +--------------+
          |
          v
        /   \
       / in   \
      < stock? >
       \     /
        \   /
   [yes] | | [no]
         v v
  +---------+   +-----------+
  | Reserve  |   | Backorder |
  | Item     |   | Item      |
  +---------+   +-----------+
         \         /
          v       v
           \     /
            \   /
             \ /
              |
              v
       +-----------+
       | Confirm   |
       | to Cust.  |
       +-----------+
              |
              v
             (●)
```

Guard conditions `[yes]` and `[no]` are mutually exclusive and cover all cases.

### 3. Parallel (Fork / Join)

Fork splits into concurrent paths. Join synchronizes them.

```
            ●
            |
            v
     +--------------+
     | Receive Order|
     +--------------+
            |
            v
   ====================        <-- FORK
      |       |       |
      v       v       v
  +------+ +------+ +-------+
  | Pack  | | Bill | | Email |
  | Items | | Cust | | Conf. |
  +------+ +------+ +-------+
      |       |       |
      v       v       v
   ====================        <-- JOIN
            |
            v
     +--------------+
     | Ship Order   |
     +--------------+
            |
            v
           (●)
```

All three middle actions execute concurrently. The join waits for all to finish.

### 4. Loop (Decision Feeding Back)

A decision node with one outgoing edge that loops back to an earlier action.

```
          ●
          |
          v
   +--------------+
   | Enter PIN    |
   +--------------+
          |
          v
        /   \
       / PIN  \
      < valid? >
       \     /
        \   /
   [no]  | |  [yes]
         | |
         | +----------->+-----------+
         |              | Grant     |
         v              | Access    |
   +-----------+        +-----------+
   | Show Error|              |
   +-----------+              v
         |                   (●)
         |
         v
       /   \
      / tries \
     < < max?  >
      \      /
       \    /
   [yes] | | [no]
         | |
         | +---->+-----------+
         |       | Lock      |
         v       | Account   |
   (loop back    +-----------+
    to Enter          |
    PIN)              v
                     (●)
```

## 🔹 Real-World Examples

### Example 1: Order Processing Workflow with Swimlanes

```
  |    Customer     |       System        |     Warehouse      |
  |                 |                     |                    |
  |       ●         |                     |                    |
  |       |         |                     |                    |
  |       v         |                     |                    |
  | +-----------+   |                     |                    |
  | | Browse    |   |                     |                    |
  | | Products  |   |                     |                    |
  | +-----------+   |                     |                    |
  |       |         |                     |                    |
  |       v         |                     |                    |
  | +-----------+   |                     |                    |
  | | Add to    |   |                     |                    |
  | | Cart      |   |                     |                    |
  | +-----------+   |                     |                    |
  |       |         |                     |                    |
  |       v         |                     |                    |
  | +-----------+   |                     |                    |
  | | Place     |---+---->+-----------+   |                    |
  | | Order     |   |     | Validate  |   |                    |
  | +-----------+   |     | Order     |   |                    |
  |                 |     +-----------+   |                    |
  |                 |          |          |                    |
  |                 |          v          |                    |
  |                 |        /   \        |                    |
  |                 |       /valid \      |                    |
  |                 |      < order? >     |                    |
  |                 |       \     /       |                    |
  |                 |        \   /        |                    |
  |                 |   [yes] | | [no]    |                    |
  |                 |         | |         |                    |
  |                 |         | +-->+---------+                |
  |                 |         |     | Notify  |                |
  |                 |         |     | Error   |                |
  |                 |         |     +---------+                |
  |                 |         |          |                     |
  |                 |         |          v                     |
  |                 |         |         (X)   <-- flow final   |
  |                 |         v               (order invalid,  |
  |                 |   +-----------+          one path ends)  |
  |                 |   | Charge    |                          |
  |                 |   | Payment   |                          |
  |                 |   +-----------+                          |
  |                 |         |                                |
  |                 |         v                                |
  |                 |  ==================    <-- FORK          |
  |                 |     |           |                        |
  |                 |     v           +---->+------------+     |
  |                 | +-----------+        | Pick &      |     |
  |                 | | Send      |        | Pack Items  |     |
  |                 | | Confirm.  |        +------------+     |
  |                 | | Email     |              |             |
  |                 | +-----------+              v             |
  |                 |     |           +------------+           |
  |                 |     |           | Ship Order  |          |
  |                 |     |           +------------+           |
  |                 |     |                  |                  |
  |                 |     v                  v                  |
  |                 |  ==================    <-- JOIN           |
  |                 |         |                                |
  | +-----------+   |         |                                |
  | | Receive   |<--+---------+                                |
  | | Package   |   |                                         |
  | +-----------+   |                                         |
  |       |         |                                         |
  |       v         |                                         |
  |      (●)        |                                         |
```

### Example 2: Login Process with Retry Loop

```
                ●
                |
                v
        +--------------+
        | Display      |
        | Login Form   |
        +--------------+
                |
                v
        +--------------+
        | Enter        |
        | Credentials  |
        +--------------+
                |
                v
        +--------------+
        | Submit       |
        | Login        |
        +--------------+
                |
                v
        +--------------+
        | Authenticate |
        | User         |
        +--------------+
                |
                v
              /   \
             /valid \
            <credentials?>
             \      /
              \    /
         [yes] |  | [no]
               |  |
               |  v
               |  +--------------+
               |  | Increment    |
               |  | Fail Counter |
               |  +--------------+
               |        |
               |        v
               |      /   \
               |     /count \
               |    < >= 3?  >
               |     \     /
               |      \   /
               |  [no] | | [yes]
               |       | |
               |       | v
               |       | +--------------+
               |       | | Lock Account |
               |       | +--------------+
               |       |       |
               |       |       v
               |       |  +--------------+
               |       |  | Show Lockout |
               |       |  | Message      |
               |       |  +--------------+
               |       |       |
               |       |       v
               |       |      (●)
               |       |
               |       v
               |  +--------------+
               |  | Show Error   |
               |  | "Invalid     |
               |  |  credentials"|
               |  +--------------+
               |       |
               |       v
               |  (loop back to
               |   "Display Login
               |   Form" above)
               |
               v
        +--------------+
        | Load User    |
        | Dashboard    |
        +--------------+
                |
                v
               (●)
```

### Example 3: Parallel Order Fulfillment (Fork/Join)

```
                    ●
                    |
                    v
            +--------------+
            | Receive      |
            | Order        |
            +--------------+
                    |
                    v
            +--------------+
            | Verify       |
            | Payment      |
            +--------------+
                    |
                    v
                  /   \
                 /paid? \
                <       >
                 \     /
                  \   /
            [yes]  | |  [no]
                   | |
                   | +---->+--------------+
                   |       | Cancel Order |
                   |       +--------------+
                   |              |
                   |              v
                   |             (●)
                   v
         ======================       <-- FORK
            |              |
            v              v
    +-----------+   +--------------+
    | Pack      |   | Generate     |
    | Items     |   | Invoice      |
    +-----------+   +--------------+
         |                |
         v                v
    +-----------+   +--------------+
    | Apply     |   | Email        |
    | Shipping  |   | Invoice to   |
    | Label     |   | Customer     |
    +-----------+   +--------------+
         |                |
         v                v
         ======================       <-- JOIN
                   |
                   v
           +--------------+
           | Hand to      |
           | Carrier      |
           +--------------+
                   |
                   v
           +--------------+         +--------------+
           | [Shipment]   |-------->| Update Order |
           | {in-transit} |         | Status       |
           +--------------+         +--------------+
            (object node)                  |
                                           v
                                     +----------\
                                     | Send      \
                                     | Tracking   >    <-- send signal
                                     | Notif.    /
                                     +----------/
                                           |
                                           v
                                          (●)
```

This example shows packing and invoicing happening **in parallel** (true concurrency via fork/join), then synchronizing before handoff to the carrier. It also demonstrates an **object node** (`[Shipment]` with state `{in-transit}`) and a **send signal** for the tracking notification.

## 🔹 Tips and Best Practices

- **Use swimlanes for multi-actor processes.** Any time more than one role or system is involved, swimlanes make responsibilities unambiguous.
- **Keep guard conditions mutually exclusive and complete.** Every decision must cover all possible outcomes. If a diamond has `[yes]` it must also have `[no]` (or `[else]`). Overlapping guards create ambiguity.
- **Always use merge nodes after decisions.** Don't let branched paths dangle. Bring them back together with an explicit merge diamond before the next shared action.
- **Distinguish activity final from flow final.** Use activity final `(circle-dot)` only when you mean "stop everything." Use flow final `(X)` when one path ends but others should continue (common in fork/join scenarios).
- **Use object nodes to show data dependencies.** When the output of one action is the input of another, make the data object explicit. This clarifies what information flows, not just what actions happen.
- **Name actions with verb phrases.** "Validate Order" not "Order Validation." "Send Email" not "Email." Actions are things the system *does*.
- **Don't overcrowd.** If a diagram has more than about 15-20 actions, consider breaking it into sub-activities (one action that references a separate activity diagram).
- **Start with the happy path.** Lay out the main success flow first, then add branches for errors and exceptions.
- **Use signals for async communication.** When two processes communicate asynchronously (e.g., sending an email notification), use send/receive signals rather than direct flow edges.
- **Pins add precision.** When you need to specify exactly what data goes in and out of an action (typed parameters), pins on the action border are cleaner than separate object nodes.

See also: [[State Machine Diagram]], [[Use Case Diagram]], [[Sequence Diagram]], [[Interaction Overview Diagram]]
