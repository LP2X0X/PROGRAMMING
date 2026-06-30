---
tags:
  - mermaid
  - state-diagram
  - diagram
---

# State Diagrams

State diagrams model the **lifecycle of an entity** -- the discrete states it can be in and the transitions (events) that move it between them. Mermaid uses `stateDiagram-v2` for the modern syntax with better rendering and feature support. Always prefer `stateDiagram-v2` over the legacy `stateDiagram` declaration.

See also: [[Syntax Basics]] | [[Styling and Themes]]

---

## 🔹 Quick Reference

| Syntax | Meaning | Example |
|---|---|---|
| `stateDiagram-v2` | Diagram declaration (always use v2) | Top-level block |
| `s1` | State with ID as label | `Idle` |
| `s1 : Description` | State with custom label | `s1 : Awaiting Input` |
| `[*] --> s1` | Start transition | `[*] --> Idle` |
| `s1 --> [*]` | End transition | `Done --> [*]` |
| `s1 --> s2` | Transition (no label) | `Idle --> Processing` |
| `s1 --> s2 : Event` | Labeled transition | `Idle --> Processing : Submit` |
| `state "Name" as s1 { }` | Composite (nested) state | Contains sub-states |
| `state fork_name <<fork>>` | Fork pseudo-state | Splits into parallel paths |
| `state join_name <<join>>` | Join pseudo-state | Synchronizes parallel paths |
| `state if_state <<choice>>` | Choice pseudo-state | Conditional branching |
| `note right of s1 : text` | Note attached to a state | Annotation on right side |
| `note left of s1 : text` | Note attached to a state | Annotation on left side |
| `--` | Concurrency separator | Divides concurrent regions inside composite state |
| `direction LR` | Layout direction | Left-to-right instead of default top-to-bottom |

---

## 🔹 Basic States and Transitions

A state is declared by simply using it in a transition, or explicitly with a label using `stateName : Label Text`. Transitions use `-->` with an optional `: Label` for the triggering event or guard condition.

- **Start** is represented by `[*]` at the source of a transition
- **End** is represented by `[*]` at the target of a transition
- You can have **multiple end states** -- `[*]` can appear as a target more than once
- State IDs **cannot contain spaces** -- use the `: Label` syntax for readable names

### Declaring States

There are two approaches:

1. **Implicit** -- just reference the state name in a transition and Mermaid creates it automatically
2. **Explicit** -- declare with `stateName : Human Readable Label` to control what text appears

### Transitions

Transitions connect states with `-->`. Add a label after `:` to describe the event or condition that triggers the transition.

```
s1 --> s2           %% unlabeled transition
s1 --> s2 : event   %% labeled transition
```

### Live Example

```mermaid
stateDiagram-v2
    [*] --> Idle
    Idle --> Processing : Submit
    Processing --> Done : Complete
    Processing --> Error : Fail
    Error --> Idle : Reset
    Done --> [*]
```

### Labeling States

Use `stateName : Label Text` to give a state a human-readable label that differs from its ID. This is especially useful when you want descriptive names with spaces.

```mermaid
stateDiagram-v2
    [*] --> s1
    s1 : Awaiting Input
    s1 --> s2 : User submits form
    s2 : Validating Data
    s2 --> s3 : Valid
    s2 --> s1 : Invalid
    s3 : Record Saved
    s3 --> [*]
```

---

## 🔹 Composite (Nested) States

Composite states contain **sub-states** -- useful for modeling phases that have their own internal lifecycle. Use the `state "Label" as id { ... }` syntax with curly braces to define the nested structure.

Key rules:
- Each composite state can have its own `[*]` start and end markers
- Sub-states inside a composite are scoped to that composite
- Transitions can go from inside a composite to outside (and vice versa)
- Keep nesting to **2 levels max** for readability; beyond that, break into separate diagrams

```mermaid
stateDiagram-v2
    [*] --> Active

    state "Active" as Active {
        [*] --> Running
        Running --> Paused : Pause
        Paused --> Running : Resume
        Running --> [*] : Complete
    }

    Active --> Cancelled : Cancel
    Active --> [*]
```

### Nesting Multiple Levels

You can nest composite states inside other composite states for deeper hierarchy.

```mermaid
stateDiagram-v2
    [*] --> Application

    state "Application" as Application {
        [*] --> Login

        state "Login" as Login {
            [*] --> EnteringCredentials
            EnteringCredentials --> Authenticating : Submit
            Authenticating --> EnteringCredentials : Fail
            Authenticating --> [*] : Success
        }

        Login --> Dashboard : Authenticated
        Dashboard --> [*] : Logout
    }

    Application --> [*]
```

---

## 🔹 Fork and Join (Parallelism)

**Fork** splits a single transition into multiple concurrent paths. **Join** synchronizes them back into one path. Declare pseudo-states with `<<fork>>` and `<<join>>`.

- Fork/join pseudo-states render as **horizontal bars**, not boxes
- All paths leaving a fork run concurrently
- A join waits for **all** incoming paths to complete before continuing
- Use descriptive names like `fork_processing` and `join_processing` for clarity

```mermaid
stateDiagram-v2
    [*] --> OrderReceived

    state fork_processing <<fork>>
    state join_processing <<join>>

    OrderReceived --> fork_processing

    fork_processing --> PickItems
    fork_processing --> ChargePayment
    fork_processing --> PrepareInvoice

    PickItems --> join_processing
    ChargePayment --> join_processing
    PrepareInvoice --> join_processing

    join_processing --> Shipped
    Shipped --> [*]
```

---

## 🔹 Choice (Conditional Branching)

Choice pseudo-states represent **decision points** where the outgoing path depends on a guard condition. Declare with `<<choice>>`. Label each outgoing transition with the condition it represents.

- A choice node is rendered as a **diamond** shape
- Each outgoing transition should have a label describing its guard condition
- Ensure all possible outcomes are covered for completeness

```mermaid
stateDiagram-v2
    [*] --> Evaluating

    state is_valid <<choice>>

    Evaluating --> is_valid : Check result
    is_valid --> Approved : Score >= 80
    is_valid --> Review : Score 50-79
    is_valid --> Rejected : Score < 50

    Approved --> [*]
    Review --> Evaluating : Re-evaluate
    Rejected --> [*]
```

---

## 🔹 Notes

Attach annotations to states using `note left of` or `note right of`. Notes provide additional context without affecting the state machine logic.

- `note right of StateName : Text` -- note on the right side
- `note left of StateName : Text` -- note on the left side
- Notes are purely informational and do not affect transitions
- Use notes to document business rules, timing constraints, or side effects

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Review : Submit
    Review --> Published : Approve
    Review --> Draft : Request changes
    Published --> [*]

    note right of Draft : Author can edit freely
    note left of Review : Reviewer checks content quality
    note right of Published : Content is live and visible to users
```

---

## 🔹 Concurrency

Inside a composite state, use `--` to separate **concurrent regions** -- independent parts of the state that operate simultaneously. Each region has its own `[*]` start and its own internal transitions.

- Concurrent regions are visually separated by a dashed line
- Each region progresses independently
- The composite state completes when **all** regions reach their end

```mermaid
stateDiagram-v2
    [*] --> Active

    state Active {
        [*] --> DownloadingData
        DownloadingData --> DataReady : Download complete

        --

        [*] --> RenderingUI
        RenderingUI --> UIReady : Render complete
    }

    Active --> [*]
```

---

## 🔹 Real-World Example: Order Processing Lifecycle

A complete order state diagram combining composite states, choice, and notes. Models the full lifecycle from placement through fulfillment, with cancellation and payment failure paths.

```mermaid
stateDiagram-v2
    [*] --> Placed

    Placed --> PaymentPending : Confirm order

    state is_paid <<choice>>
    PaymentPending --> is_paid : Process payment
    is_paid --> Paid : Payment success
    is_paid --> PaymentFailed : Payment declined

    PaymentFailed --> PaymentPending : Retry payment
    PaymentFailed --> Cancelled : Max retries exceeded

    state "Fulfillment" as Fulfillment {
        [*] --> Picking
        Picking --> Packing : Items picked
        Packing --> ReadyToShip : Packed and labeled
    }

    Paid --> Fulfillment
    Fulfillment --> Shipped : Dispatched with carrier
    Shipped --> Delivered : Delivery confirmed
    Delivered --> [*]
    Cancelled --> [*]

    Placed --> Cancelled : Customer cancels

    note right of Placed : Customer sees order confirmation page
    note left of Cancelled : Refund issued automatically if paid
    note right of Shipped : Tracking number sent to customer
```

---

## 🔹 Real-World Example: TCP Connection State Machine

Models the standard TCP connection lifecycle including the three-way handshake, data transfer, and connection teardown.

```mermaid
stateDiagram-v2
    [*] --> CLOSED

    CLOSED --> LISTEN : Passive open
    CLOSED --> SYN_SENT : Active open / Send SYN

    LISTEN --> SYN_RECEIVED : Recv SYN / Send SYN+ACK
    LISTEN --> CLOSED : Close

    SYN_SENT --> ESTABLISHED : Recv SYN+ACK / Send ACK
    SYN_SENT --> CLOSED : Timeout

    SYN_RECEIVED --> ESTABLISHED : Recv ACK
    SYN_RECEIVED --> CLOSED : Timeout

    ESTABLISHED --> FIN_WAIT_1 : Close / Send FIN
    ESTABLISHED --> CLOSE_WAIT : Recv FIN / Send ACK

    FIN_WAIT_1 --> FIN_WAIT_2 : Recv ACK
    FIN_WAIT_1 --> CLOSING : Recv FIN / Send ACK
    FIN_WAIT_1 --> TIME_WAIT : Recv FIN+ACK / Send ACK

    FIN_WAIT_2 --> TIME_WAIT : Recv FIN / Send ACK

    CLOSING --> TIME_WAIT : Recv ACK

    CLOSE_WAIT --> LAST_ACK : Close / Send FIN

    LAST_ACK --> CLOSED : Recv ACK

    TIME_WAIT --> CLOSED : 2MSL timeout

    note right of ESTABLISHED : Data transfer occurs here
    note left of TIME_WAIT : Waits 2x max segment lifetime
    note right of CLOSED : No connection exists
```

---

## 🔹 Real-World Example: Door Lock State Machine

A simple embedded system state machine modeling a door lock with PIN entry and auto-lock behavior.

```mermaid
stateDiagram-v2
    [*] --> Locked

    Locked --> EnteringPIN : Keypad pressed

    state check_pin <<choice>>
    EnteringPIN --> check_pin : PIN submitted
    check_pin --> Unlocked : Correct PIN
    check_pin --> Locked : Wrong PIN (attempts < 3)
    check_pin --> Alarming : Wrong PIN (3rd attempt)

    Unlocked --> Open : Handle turned
    Unlocked --> Locked : Auto-lock (30s timeout)
    Open --> Unlocked : Door closed
    Open --> Locked : Door closed + auto-lock

    Alarming --> Locked : Admin reset
    Alarming --> Locked : Alarm timeout (5 min)

    note right of Locked : LED shows red
    note right of Unlocked : LED shows green, deadbolt retracted
    note left of Alarming : Buzzer sounds, alert sent to owner
    note right of Open : Door sensor detects open position
```

---

## 🔹 Tips and Gotchas

- Always use `stateDiagram-v2` -- the original `stateDiagram` has fewer features and worse rendering
- State IDs **cannot contain spaces** -- use the `s1 : Label with spaces` syntax for readable labels
- `[*]` can appear multiple times -- you can have multiple start contexts (inside composites) and multiple end transitions
- Composite states must use the `state "Label" as id { }` syntax with curly braces
- Fork/join pseudo-states render as **horizontal bars**, not visible boxes
- Choice pseudo-states render as **diamonds**
- Keep nesting to **2 levels max** for readability; beyond that, consider separate linked diagrams
- If a diagram gets complex, split into multiple diagrams and link with [[wiki links]]
- Use `direction LR` inside the diagram block to switch to left-to-right layout
- Comments use `%%` syntax: `%% this is a comment`
- Transition labels should describe the **event** or **condition**, not the action taken

---

**See also:** [[Syntax Basics]] | [[Flowcharts]] | [[Sequence Diagrams]] | [[Styling and Themes]]
