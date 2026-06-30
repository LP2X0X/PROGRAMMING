---
tags:
  - mermaid
  - flowchart
  - diagram
---

# Flowcharts

Flowcharts are the most commonly used Mermaid diagram. They represent processes, decision trees, system architectures, and any flow of steps or data. Use `flowchart` (preferred) or `graph` as the declaration keyword.

See also: [[Syntax Basics]] for general rules, [[Styling and Themes]] for customization.

---

## 🔹 Quick Reference: Node Shapes

| Shape | Syntax | Renders As |
|---|---|---|
| Rectangle | `A[Text]` | Standard process box |
| Rounded rectangle | `A(Text)` | Rounded corners |
| Stadium / pill | `A([Text])` | Fully rounded ends |
| Subroutine | `A[[Text]]` | Double-bordered box |
| Cylinder | `A[(Text)]` | Database symbol |
| Circle | `A((Text))` | Circular node |
| Asymmetric / flag | `A>Text]` | Flag/ribbon shape |
| Diamond / decision | `A{Text}` | Rhombus for decisions |
| Hexagon | `A{{Text}}` | Six-sided shape |
| Parallelogram | `A[/Text/]` | Input/output shape |
| Parallelogram alt | `A[\Text\]` | Reversed lean |
| Trapezoid | `A[/Text\]` | Wider at top |
| Trapezoid alt | `A[\Text/]` | Wider at bottom |
| Double circle | `A(((Text)))` | Double ring |

## 🔹 Quick Reference: Link Types

| Link | Syntax | Description |
|---|---|---|
| Arrow | `A --> B` | Standard directional |
| Open link | `A --- B` | No arrowhead |
| Text on arrow | `A -->\|text\| B` | Label on link |
| Text on arrow (alt) | `A -- text --> B` | Same result, alternate syntax |
| Dotted arrow | `A -.-> B` | Dashed line with arrow |
| Dotted open | `A -.- B` | Dashed line, no arrow |
| Dotted with text | `A -. text .-> B` | Dashed + label |
| Thick arrow | `A ==> B` | Bold/thick line with arrow |
| Thick open | `A === B` | Bold line, no arrow |
| Thick with text | `A == text ==> B` | Bold + label |
| Multidirectional | `A <--> B` | Arrows on both ends |

---

## 🔹 Node Shapes in Detail

Each node shape communicates a different meaning. Here they are grouped with live examples:

### Basic Shapes

```mermaid
flowchart LR
    rect[Rectangle]
    round(Rounded)
    stadium([Stadium])
    sub[[Subroutine]]
    cyl[(Cylinder)]
    circ((Circle))
```

### Decision and Special Shapes

```mermaid
flowchart LR
    diamond{Decision}
    hex{{Hexagon}}
    para[/Parallelogram/]
    para2[\Parallelogram Alt\]
    trap[/Trapezoid\]
    trap2[\Trapezoid Alt/]
    flag>Flag Shape]
    dblcirc(((Double Circle)))
```

> [!tip] When to Use Which Shape
> - **Rectangle** `[Text]` — generic process step (the default workhorse)
> - **Diamond** `{Text}` — decision/branch point (yes/no, if/else)
> - **Stadium** `([Text])` — start/end terminal
> - **Cylinder** `[(Text)]` — database or data store
> - **Parallelogram** `[/Text/]` — input/output (user input, display)
> - **Circle** `((Text))` — connector or junction point
> - **Subroutine** `[[Text]]` — call to external process or function
> - **Hexagon** `{{Text}}` — preparation or condition setup

---

## 🔹 Link Types in Detail

### All Arrow Styles

```mermaid
flowchart LR
    A1[A] --> B1[B]
    A2[A] --- B2[B]
    A3[A] -.-> B3[B]
    A4[A] ==> B4[B]
    A5[A] <--> B5[B]
    A6[A] -.- B6[B]
    A7[A] === B7[B]
```

### Links with Text Labels

```mermaid
flowchart LR
    A -->|arrow label| B
    C -- text --> D
    E -.->|dotted label| F
    G ==>|thick label| H
```

> [!info] Two Syntaxes for Link Text
> Both produce identical results — choose whichever reads better:
> - Pipe syntax: `A -->|label| B`
> - Inline syntax: `A -- label --> B`

### Chaining Links

You can chain multiple connections on one line:

```mermaid
flowchart LR
    A --> B --> C --> D
    X --> Y & Z
    P & Q --> R
```

- `A --> B --> C` chains sequentially
- `X --> Y & Z` means X connects to **both** Y and Z
- `P & Q --> R` means **both** P and Q connect to R

---

## 🔹 Subgraphs

Subgraphs group related nodes into a labeled box. They are essential for organizing complex diagrams.

### Basic Subgraph

```mermaid
flowchart TB
    subgraph Frontend
        A[React App]
        B[API Client]
    end
    subgraph Backend
        C[REST API]
        D[Auth Service]
    end
    subgraph Database
        E[(PostgreSQL)]
    end
    A --> B
    B --> C
    C --> D
    C --> E
```

### Direction Per Subgraph

Each subgraph can have its own flow direction using the `direction` keyword:

```mermaid
flowchart LR
    subgraph Frontend
        direction TB
        A[UI] --> B[State]
        B --> C[Components]
    end
    subgraph Backend
        direction TB
        D[Controller] --> E[Service]
        E --> F[Repository]
    end
    C --> D
```

### Nested Subgraphs

Subgraphs can nest inside each other:

```mermaid
flowchart TB
    subgraph Cloud
        subgraph Region-US
            A[Server 1]
            B[Server 2]
        end
        subgraph Region-EU
            C[Server 3]
        end
    end
    A --> B
    B --> C
```

> [!warning] Common Mistake
> - Every `subgraph` must have a matching `end`
> - Don't forget the title after `subgraph` — omitting it can cause parse errors
> - `direction` only works with the `flowchart` declaration, **not** `graph`

---

## 🔹 Styling Nodes and Links

### Inline Style on a Node

```mermaid
flowchart LR
    A[Error]:::errorStyle --> B[OK]:::okStyle
    classDef errorStyle fill:#ff6b6b,stroke:#c0392b,color:#fff
    classDef okStyle fill:#51cf66,stroke:#2f9e44,color:#fff
```

### Using `style` Directly

```mermaid
flowchart LR
    A[Styled Node] --> B[Default Node]
    style A fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f
```

### Style Properties

| Property | Example | Description |
|---|---|---|
| `fill` | `fill:#f9f` | Background color |
| `stroke` | `stroke:#333` | Border color |
| `stroke-width` | `stroke-width:2px` | Border thickness |
| `color` | `color:#fff` | Text color |
| `stroke-dasharray` | `stroke-dasharray: 5 5` | Dashed border |

### Class Definitions

Define reusable styles with `classDef` and apply with `:::`:

```
classDef myClass fill:#f96,stroke:#333,stroke-width:2px
A[Node]:::myClass
```

> [!tip] Tip
> Use `classDef default` to style ALL nodes at once:
> `classDef default fill:#f9f9f9,stroke:#333,stroke-width:1px`

See [[Styling and Themes]] for comprehensive styling and theming options.

---

## 🔹 Real-World Example: Decision Flowchart

```mermaid
flowchart TD
    Start([Start]) --> Input[/Get User Input/]
    Input --> Validate{Valid?}
    Validate -->|Yes| Process[Process Data]
    Validate -->|No| Error[Show Error]
    Error --> Input
    Process --> Save[(Save to DB)]
    Save --> Success{Saved OK?}
    Success -->|Yes| Done([Done])
    Success -->|No| Retry{Retries Left?}
    Retry -->|Yes| Save
    Retry -->|No| Fail([Fail])
```

---

## 🔹 Real-World Example: CI/CD Pipeline

```mermaid
flowchart LR
    subgraph Trigger
        A([Push to main])
    end
    subgraph Build
        B[Restore Packages]
        C[Compile]
        D[Run Unit Tests]
    end
    subgraph Quality
        E[Code Analysis]
        F[Integration Tests]
    end
    subgraph Deploy
        G{Tests Pass?}
        H[Build Docker Image]
        I[Push to Registry]
        J[Deploy to Staging]
        K{Approval?}
        L[Deploy to Production]
    end
    A --> B --> C --> D
    D --> E & F
    E & F --> G
    G -->|Yes| H --> I --> J --> K
    G -->|No| M([Notify Team])
    K -->|Approved| L
    K -->|Rejected| N([Rollback])
```

---

## 🔹 Real-World Example: System Architecture

```mermaid
flowchart TB
    subgraph Client
        direction LR
        Browser[Browser]
        Mobile[Mobile App]
    end
    subgraph API Gateway
        LB([Load Balancer])
    end
    subgraph Services
        Auth[Auth Service]
        Users[User Service]
        Orders[Order Service]
        Notify[Notification Service]
    end
    subgraph Data
        direction LR
        DB1[(User DB)]
        DB2[(Order DB)]
        Cache[(Redis Cache)]
        Queue[Message Queue]
    end
    Browser & Mobile --> LB
    LB --> Auth & Users & Orders
    Auth --> DB1
    Users --> DB1
    Users --> Cache
    Orders --> DB2
    Orders --> Queue
    Queue --> Notify
```

---

## 🔹 Real-World Example: Error Handling Flow

```mermaid
flowchart TD
    Request([HTTP Request]) --> Auth{Authenticated?}
    Auth -->|No| R401[Return 401]
    Auth -->|Yes| Perm{Authorized?}
    Perm -->|No| R403[Return 403]
    Perm -->|Yes| Parse{Valid Body?}
    Parse -->|No| R400[Return 400]
    Parse -->|Yes| Execute[Execute Logic]
    Execute --> Result{Success?}
    Result -->|Yes| R200[Return 200]
    Result -->|No| R500[Return 500]
    R500 --> Log[(Log Error)]
```

---

## 🔹 Common Patterns and Tips

### Interaction / Click Events

You can make nodes clickable (opens URLs in the browser):

```
click A "https://example.com" "Open docs" _blank
```

### Long Labels

For multiline labels, use `<br/>` inside quoted text:

```mermaid
flowchart LR
    A["Line 1<br/>Line 2<br/>Line 3"] --> B[Short]
```

### Node ID Reuse

Once you define a node with a label, reuse just the ID:

```mermaid
flowchart LR
    myNode[This is my node] --> B[Next]
    B --> myNode
```

The label `This is my node` is only written once. Every subsequent reference to `myNode` uses the same node.

> [!warning] Common Mistake
> If you write `myNode[Label 1]` in one place and `myNode[Label 2]` elsewhere, the last label wins. Always define the label once, then use just the ID.

### Invisible Links for Layout

Sometimes you need to nudge layout without a visible connection. Use a link styled invisible:

```
A ~~~ B
```

This creates an invisible link that affects positioning without drawing anything.

---

**See also**: [[Syntax Basics]] | [[Subgraphs in Flowcharts]] | [[Styling and Themes]] | [[Sequence Diagrams]]
