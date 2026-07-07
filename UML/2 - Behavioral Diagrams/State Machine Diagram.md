---
tags:
  - uml
  - state-machine-diagram
  - behavioral
---

# State Machine Diagram

A **state machine diagram** (also called a **statechart diagram**) models the discrete **states** an object passes through during its lifetime and the **transitions** triggered by events. It answers: *"What states can this object be in, and what causes it to change state?"*

> [!tip] Use state machine diagrams for objects with **complex lifecycle behavior** — orders, connections, UI components, devices — anything where the object's response to an event depends on its current state.

---

## 🔹 State Machine vs Activity Diagram

| Aspect                | State Machine Diagram                | [[Activity Diagram]]                |
| --------------------- | ------------------------------------ | ----------------------------------- |
| **Focus**             | States of a single object            | Flow of activities / workflow       |
| **Driven by**         | Events (reactive)                    | Control flow (procedural)           |
| **Shows**             | What state am I in?                  | What step am I doing?              |
| **Parallelism**       | Fork/join (orthogonal regions)       | Fork/join (concurrent activities)   |
| **Best for**          | Object lifecycle, protocols          | Business processes, algorithms      |
| **Nodes represent**   | States (waiting for something)       | Actions (doing something)           |

**Rule of thumb**: If the object *waits* for events and *reacts* differently depending on its current state → state machine. If you're modeling a *workflow* with a clear start-to-finish sequence → activity diagram.

---

## 🔹 Core Notation

### State

A rounded rectangle with up to three compartments.

```
  ┌─────────────────────────┐
  │       Processing        │   ← state name
  ├─────────────────────────┤
  │ entry / startTimer()    │   ← entry action (runs on entering)
  │ do / processItems()     │   ← do activity (runs while in state)
  │ exit / stopTimer()      │   ← exit action (runs on leaving)
  └─────────────────────────┘
```

- **entry/** — executes every time the state is entered (cannot be skipped)
- **do/** — ongoing activity that runs for the duration of the state (can be interrupted)
- **exit/** — executes every time the state is left (cannot be skipped)

> [!info] You don't need all three compartments. Most states in practice just have a name.

### Initial and Final States

```
  ●                         ◉
  (filled circle)           (filled circle
   = initial state           inside circle
                              = final state)

  ● ──────> ┌─────────┐ ──────> ◉
             │  Idle   │
             └─────────┘
```

- **Initial pseudo-state** (`●`): exactly one per region, with exactly one outgoing transition (no trigger)
- **Final state** (`◉`): object's lifecycle ends; can have zero or more

---

## 🔹 Transitions

A transition is an arrow from one state to another, labeled with:

```
  trigger [guard] / action
```

```
  ┌──────────┐  paymentReceived [amount >= total] / sendConfirmation()  ┌──────────┐
  │ Pending  │ ──────────────────────────────────────────────────────>  │  Paid    │
  └──────────┘                                                         └──────────┘
```

| Component     | Required? | Meaning                                          |
| ------------- | --------- | ------------------------------------------------ |
| **trigger**   | Optional  | Event that fires the transition                  |
| **[guard]**   | Optional  | Boolean condition — transition only if true       |
| **/action**   | Optional  | Effect executed during the transition             |

- If **no trigger**: the transition fires as soon as the state's do-activity completes (completion transition)
- Multiple transitions from the same state with the same trigger but different guards = **conditional branching**

---

## 🔹 Internal vs Self-Transitions

```
  ┌───────────────────────┐             ┌───────────┐
  │       Active          │             │   Active   │──┐
  ├───────────────────────┤             └───────────┘  │ keyPress / beep()
  │ keyPress / beep()     │                    ▲───────┘
  │  (internal transition)│              (self-transition)
  └───────────────────────┘
```

| Type                    | Entry/Exit Actions Run? | State Changed? |
| ----------------------- | :---------------------: | :------------: |
| **Internal transition** |           No            |       No       |
| **Self-transition**     |           Yes           |   No (same)    |

- **Internal transition**: handles the event *without leaving the state* — entry/exit do NOT fire
- **Self-transition**: the state is exited and re-entered — entry/exit actions DO fire

> [!warning] This distinction matters. If your entry action initializes a timer, a self-transition resets it; an internal transition does not.

---

## 🔹 Composite (Nested) States

A state that contains an entire sub-state machine.

```
  ┌──────────────────────────────────────────┐
  │              Processing                   │
  │                                           │
  │   ● ──> ┌──────────┐ ──> ┌──────────┐   │
  │         │ Validating│     │ Saving   │   │
  │         └──────────┘     └──────────┘   │
  │                               │          │
  │                               ▼          │
  │                              ◉           │
  └──────────────────────────────────────────┘
                    │
                    │ cancel / rollback()
                    ▼
              ┌──────────┐
              │ Cancelled │
              └──────────┘
```

- A transition **from the outer state boundary** applies to ALL sub-states (like `cancel` above — works whether in Validating or Saving)
- Sub-states can have their own initial/final states
- When the sub-machine reaches its final state, a **completion transition** from the composite state fires

### History States

When re-entering a composite state, should it start from the beginning or resume where it left off?

```
  (H)  = shallow history — resumes the last active sub-state (top level only)
  (H*) = deep history — resumes the exact nested sub-state at any depth
```

```
  ┌───────────────────────────┐
  │        Playback           │
  │                           │
  │  (H) ──> ┌──────────┐   │
  │           │ Playing  │   │
  │           └──────────┘   │
  │           ┌──────────┐   │
  │           │ Paused   │   │
  │           └──────────┘   │
  └───────────────────────────┘
```

If you leave Playback while in "Paused" and later re-enter via the `(H)` pseudo-state, you resume in "Paused" — not the initial state.

---

## 🔹 Pseudo-States

### Choice Pseudo-State

A diamond that evaluates guards *after* arriving (dynamic branching).

```
                    ┌──────────┐
                    │ Evaluating│
                    └─────┬────┘
                          ▼
                        ◇
                       / \
      [score >= 70]  /     \  [score < 70]
                   /         \
                  ▼           ▼
           ┌────────┐   ┌────────┐
           │ Passed │   │ Failed │
           └────────┘   └────────┘
```

> [!info] Choice vs Junction
> A **junction** (`●` small filled circle) evaluates guards *statically* before the transition fires. A **choice** (`◇` diamond) evaluates guards *dynamically* after arriving. In practice, the difference rarely matters — use choice when the guard depends on something computed during the transition.

### Fork and Join

For **concurrent (orthogonal) regions** — the object is in multiple sub-states simultaneously.

```
             ┌──────────┐
             │  Active   │
             └─────┬─────┘
                   ▼
            ═══════════════  (fork bar)
           /                \
          ▼                  ▼
  ┌─────────────┐   ┌─────────────┐
  │ Region 1    │   │ Region 2    │
  │ ┌─────────┐ │   │ ┌─────────┐ │
  │ │Heating  │ │   │ │Spinning │ │
  │ └─────────┘ │   │ └─────────┘ │
  └─────────────┘   └─────────────┘
          \                 /
           ▼               ▼
            ═══════════════  (join bar)
                   ▼
             ┌──────────┐
             │  Done     │
             └──────────┘
```

- **Fork**: one incoming transition splits into multiple concurrent regions
- **Join**: all regions must reach the join bar before the outgoing transition fires
- Both are drawn as thick bars (same as activity diagram notation)

---

## 🔹 Example: Order Lifecycle

```
  ●
  │
  ▼
┌───────────┐  pay [paymentValid]  ┌───────────┐   ship    ┌───────────┐
│  Created  │ ───────────────────> │   Paid    │ ────────> │  Shipped  │
└───────────┘                      └───────────┘           └─────┬─────┘
      │                                  │                       │
      │ cancel                           │ cancel                │ deliver
      │                                  │                       ▼
      │                                  │                 ┌───────────┐
      ▼                                  ▼                 │ Delivered │
┌───────────┐                      ┌───────────┐          └─────┬─────┘
│ Cancelled │                      │ Refunding │                │
└─────┬─────┘                      └─────┬─────┘                ▼
      ▼                                  ▼                     ◉
     ◉                                  ◉
```

---

## 🔹 Example: TCP Connection (Simplified)

```
  ●
  │
  ▼
┌──────────┐  connect / SYN   ┌──────────────┐  SYN+ACK / ACK   ┌──────────────┐
│  Closed  │ ───────────────> │  SYN Sent    │ ───────────────>  │ Established  │
└──────────┘                  └──────────────┘                   └──────┬───────┘
      ▲                                                                │
      │                            FIN+ACK / ACK                       │ close / FIN
      │                        ┌──────────────┐                        │
      └─────── timeout ─────── │  FIN Wait    │ <─────────────────────┘
                               └──────────────┘
```

---

## 🔹 Example: Elevator

```
  ●
  │
  ▼
┌──────────────┐
│     Idle     │
│ entry/       │
│  closeDoors()│
└──────┬───────┘
       │ floorRequested(n) [n != currentFloor]
       ▼
     ◇ choice
    /         \
[n > current]  [n < current]
  /               \
 ▼                 ▼
┌───────────┐  ┌──────────────┐
│ Moving Up │  │ Moving Down  │
│ do/       │  │ do/          │
│  moveUp() │  │  moveDown()  │
└─────┬─────┘  └──────┬───────┘
      │               │
      │  arrived      │ arrived
      ▼               ▼
┌──────────────────────┐
│      Stopped         │
│ entry / openDoors()  │
│ do / waitForRiders() │
│ exit / closeDoors()  │
└──────────┬───────────┘
           │ timeout [noMoreRequests]
           ▼
        ┌──────┐
        │ Idle │
        └──────┘
```

---

## 🔹 Quick Reference

| Symbol         | Meaning                                   |
| -------------- | ----------------------------------------- |
| `●`            | Initial pseudo-state                      |
| `◉`            | Final state                               |
| Rounded rect   | State                                     |
| Arrow          | Transition                                |
| `◇`            | Choice pseudo-state                       |
| `(H)` / `(H*)` | Shallow / deep history pseudo-state       |
| Thick bar      | Fork or join (concurrent)                 |
| Nested rect    | Composite (nested) state                  |
| Dashed line    | Divides orthogonal regions within a state |

---

See also: [[Activity Diagram]], [[Sequence Diagram]], [[Use Case Diagram]]
