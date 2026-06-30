---
tags:
  - mermaid
  - diagram
  - reference
---

# Other Diagrams

Mermaid supports many diagram types beyond the core set of [[Flowcharts]], sequence diagrams, class diagrams, [[State Diagrams]], [[ER Diagrams]], and [[Gantt Charts]]. This note covers the additional types -- each useful for specific visualization needs but encountered less frequently.

See also: [[Syntax Basics]] and [[Styling and Themes]].

---

## 🔹 Pie Chart

Pie charts show **proportional data** as slices of a circle. Simple but effective for quick visual breakdowns of how parts relate to a whole.

**Declaration keyword:** `pie`

### Syntax

| Syntax | Meaning |
|---|---|
| `pie` | Diagram declaration |
| `title "Chart Title"` | Optional title |
| `showData` | Display numeric values on each slice |
| `"Label" : value` | A slice with its label and numeric value |

### Key Points

- Values are automatically converted to percentages relative to the total.
- No limit on slices, but beyond 6-7 the chart becomes hard to read.
- Colors are assigned automatically from the theme palette.
- `showData` prints the raw value alongside each slice.

### Example: Programming Language Popularity

```mermaid
pie showData
    title Languages Used in Project
    "C#" : 45
    "TypeScript" : 25
    "SQL" : 15
    "Python" : 10
    "Other" : 5
```

### Example: Budget Breakdown

```mermaid
pie
    title Q2 Budget Allocation
    "Engineering" : 40
    "Marketing" : 20
    "Operations" : 15
    "Sales" : 15
    "R&D" : 10
```

---

## 🔹 Mindmap

Mindmaps visualize **hierarchical ideas** radiating from a central concept. The structure is defined purely by **indentation** -- each level of indentation creates a child node.

**Declaration keyword:** `mindmap`

### Syntax

| Syntax | Meaning |
|---|---|
| `mindmap` | Diagram declaration |
| Root text (no indent) | Central/root node |
| 2-space indent | Child node |
| 4-space indent | Grandchild node |
| `(Round edges)` | Rounded rectangle shape |
| `[Square]` | Square/box shape |
| `((Circle))` | Circle shape |
| `))Bang((` | Bang/explosion shape |
| `{{Hexagon}}` | Hexagon shape |
| `)Cloud(` | Cloud shape |

### Key Points

- Indentation must be consistent (use spaces, not tabs).
- Node shapes are optional -- plain text uses the default shape.
- Keep trees reasonably balanced; heavily lopsided trees render poorly.

### Example: Study Topic Breakdown

```mermaid
mindmap
  root((C# Mastery))
    Language Fundamentals
      Types and Memory
        Value Types
        Reference Types
        Span and Memory
      Control Flow
      Pattern Matching
    OOP
      Inheritance
      Interfaces
      Polymorphism
    Advanced Topics
      Async/Await
      LINQ
      Generics
      Delegates and Events
    Ecosystem
      .NET Runtime
      NuGet
      Testing Frameworks
```

### Example: Project Architecture

```mermaid
mindmap
  root((Web App))
    Frontend
      React
        Components
        Hooks
        State Management
      Styling
        CSS Modules
        Tailwind
    Backend
      API Layer
        REST Endpoints
        Middleware
      Database
        MariaDB
        Migrations
    DevOps
      CI/CD
      Docker
      Monitoring
```

---

## 🔹 Timeline

Timelines show **events organized chronologically** within periods or sections. Good for project histories, roadmaps, and historical overviews.

**Declaration keyword:** `timeline`

### Syntax

| Syntax | Meaning |
|---|---|
| `timeline` | Diagram declaration |
| `title Text` | Timeline title |
| `section Period` | A time period grouping |
| Indented text | Events within the period |
| `Event : Description` | An event with optional description text |

### Key Points

- Each `section` groups events under a time period label.
- Events are simple text -- no complex formatting inside them.
- Multiple events in a section render as stacked items.

### Example: Technology Milestones

```mermaid
timeline
    title Major Technology Milestones
    section 1990s
        World Wide Web : Tim Berners-Lee creates HTML and HTTP
        Java : Write once, run anywhere
        JavaScript : Netscape ships browser scripting
    section 2000s
        .NET Framework : Microsoft's managed runtime
        Git : Distributed version control by Linus Torvalds
        iPhone : Mobile computing revolution
    section 2010s
        Docker : Containerization goes mainstream
        TypeScript : Typed JavaScript by Microsoft
        Kubernetes : Container orchestration
    section 2020s
        GitHub Copilot : AI-assisted coding
        ChatGPT : Large language models go mainstream
```

### Example: Product Roadmap

```mermaid
timeline
    title Product Roadmap 2026
    section Q1
        MVP Launch : Core features released
        Beta testing : 500 users onboarded
    section Q2
        Public launch : Marketing campaign
        API release : Third-party integrations
    section Q3
        Enterprise tier : SSO and audit logs
        Analytics dashboard : Usage metrics
    section Q4
        International : Multi-language support
        Marketplace : Plugin ecosystem
```

---

## 🔹 Git Graph

Git graphs visualize **branching, committing, and merging** workflows. Ideal for documenting Git strategies, explaining branching models, and illustrating release processes.

**Declaration keyword:** `gitGraph`

### Syntax

| Syntax | Meaning |
|---|---|
| `gitGraph` | Diagram declaration |
| `commit` | Add a commit to the current branch |
| `commit id: "msg"` | Commit with a custom label |
| `commit tag: "v1.0"` | Commit with a tag |
| `commit type: HIGHLIGHT` | Highlighted commit (also `REVERSE`, `NORMAL`) |
| `branch name` | Create a new branch |
| `checkout name` | Switch to a branch |
| `merge name` | Merge a branch into the current branch |
| `cherry-pick id: "abc"` | Cherry-pick a specific commit by its `id` |

### Key Points

- Branches render as colored parallel tracks.
- The `id` on commits is optional but required for cherry-pick references and labeling.
- `type: HIGHLIGHT` renders the commit node differently (filled vs outlined), useful for marking important commits.
- Keep graphs small -- beyond 15-20 commits, readability drops.

### Example: Feature Branch Workflow

```mermaid
gitGraph
    commit id: "init"
    commit id: "setup"

    branch feature/auth
    checkout feature/auth
    commit id: "add login"
    commit id: "add logout" type: HIGHLIGHT

    checkout main
    commit id: "hotfix" type: REVERSE

    branch feature/dashboard
    checkout feature/dashboard
    commit id: "layout"
    commit id: "charts"

    checkout main
    merge feature/auth

    checkout feature/dashboard
    commit id: "polish"

    checkout main
    merge feature/dashboard
    commit id: "release" tag: "v1.0"
```

### Example: Gitflow Model

```mermaid
gitGraph
    commit id: "initial"
    branch develop
    checkout develop
    commit id: "dev setup"

    branch feature/login
    commit id: "login form"
    commit id: "auth logic"
    checkout develop
    merge feature/login

    branch release/1.0
    commit id: "bump version"
    commit id: "fix typo"
    checkout main
    merge release/1.0 tag: "v1.0"
    checkout develop
    merge release/1.0

    commit id: "next feature"
```

---

## 🔹 Quadrant Chart

Quadrant charts plot items on a **2x2 matrix** with labeled axes. Useful for prioritization matrices, risk assessment, strategic planning, and any two-dimensional categorization.

**Declaration keyword:** `quadrantChart`

### Syntax

| Syntax | Meaning |
|---|---|
| `quadrantChart` | Diagram declaration |
| `title Text` | Chart title |
| `x-axis Label --> Label` | X-axis with low-end and high-end labels |
| `y-axis Label --> Label` | Y-axis with low-end and high-end labels |
| `quadrant-1 Label` | Top-right quadrant label |
| `quadrant-2 Label` | Top-left quadrant label |
| `quadrant-3 Label` | Bottom-left quadrant label |
| `quadrant-4 Label` | Bottom-right quadrant label |
| `Item: [x, y]` | Plot a point (values from 0.0 to 1.0) |

### Key Points

- Coordinates range from `[0.0, 0.0]` (bottom-left origin) to `[1.0, 1.0]` (top-right).
- Quadrant numbering is **counter-intuitive**: 1 = top-right, 2 = top-left, 3 = bottom-left, 4 = bottom-right (counter-clockwise from top-right).
- Keep to 8-12 points max for clarity.

### Example: Effort vs Impact Matrix

```mermaid
quadrantChart
    title Feature Priority Matrix
    x-axis Low Effort --> High Effort
    y-axis Low Impact --> High Impact
    quadrant-1 Plan carefully
    quadrant-2 Do first
    quadrant-3 Deprioritize
    quadrant-4 Quick wins

    Dark mode: [0.2, 0.3]
    SSO integration: [0.7, 0.85]
    Search feature: [0.35, 0.75]
    Email templates: [0.15, 0.6]
    Data export: [0.5, 0.4]
    Mobile app: [0.9, 0.9]
    Tooltip improvements: [0.1, 0.15]
```

### Example: Eisenhower Matrix

```mermaid
quadrantChart
    title Eisenhower Decision Matrix
    x-axis Not Urgent --> Urgent
    y-axis Not Important --> Important
    quadrant-1 Do immediately
    quadrant-2 Schedule it
    quadrant-3 Eliminate
    quadrant-4 Delegate

    Production bug: [0.9, 0.95]
    Code review: [0.6, 0.7]
    Team lunch planning: [0.7, 0.2]
    Learning new framework: [0.2, 0.65]
    Refactor auth module: [0.3, 0.8]
    Update README: [0.15, 0.25]
    Reply to emails: [0.8, 0.3]
    Sprint retrospective: [0.5, 0.55]
```

---

## 🔹 XY Chart (Bar / Line)

XY charts render **bar charts and line charts** on a standard Cartesian plane. Useful for plotting numeric data, trends, and comparisons.

**Declaration keyword:** `xychart-beta`

> **Note:** This diagram type is currently in beta. Syntax may change in future Mermaid versions.

### Syntax

| Syntax | Meaning |
|---|---|
| `xychart-beta` | Diagram declaration |
| `title "Text"` | Chart title |
| `x-axis "Label" [val1, val2, ...]` | X-axis label and category values |
| `x-axis "Label" min --> max` | X-axis with numeric range |
| `y-axis "Label" min --> max` | Y-axis label and numeric range |
| `bar [v1, v2, v3, ...]` | Bar series data |
| `line [v1, v2, v3, ...]` | Line series data |

### Key Points

- You can combine `bar` and `line` series in the same chart.
- X-axis can use category labels (list of strings) or numeric ranges.
- Multiple `bar` or `line` declarations will overlay series.

### Example: Monthly Revenue with Trend

```mermaid
xychart-beta
    title "Monthly Revenue (thousands)"
    x-axis [Jan, Feb, Mar, Apr, May, Jun, Jul, Aug, Sep, Oct, Nov, Dec]
    y-axis "Revenue ($k)" 0 --> 120
    bar [45, 52, 48, 61, 55, 70, 65, 78, 82, 90, 85, 105]
    line [45, 50, 53, 57, 60, 64, 68, 72, 76, 82, 88, 95]
```

### Example: Test Coverage Over Sprints

```mermaid
xychart-beta
    title "Test Coverage by Sprint"
    x-axis [S1, S2, S3, S4, S5, S6, S7, S8]
    y-axis "Coverage (%)" 0 --> 100
    line [42, 48, 55, 60, 63, 71, 75, 82]
    bar [42, 48, 55, 60, 63, 71, 75, 82]
```

---

## 🔹 Block Diagram

Block diagrams represent **system components and their connections** using a column-based layout. Useful for architecture overviews, system design, and component relationships.

**Declaration keyword:** `block-beta`

> **Note:** This diagram type is currently in beta. Syntax may change in future Mermaid versions.

### Syntax

| Syntax | Meaning |
|---|---|
| `block-beta` | Diagram declaration |
| `columns N` | Set the number of columns in the grid |
| `id["Label"]` | A block with an ID and display label |
| `id["Label"]:N` | A block spanning N columns |
| `space` | An empty cell in the grid |
| `id --> id2` | Arrow connecting two blocks |
| `block:id` / `end` | A nested container block |

### Key Points

- Layout is grid-based -- `columns` sets how many columns each row has.
- Blocks automatically flow left-to-right, top-to-bottom.
- Use `space` to skip cells and control positioning.
- Nested `block:id` / `end` pairs create grouped sub-layouts.

### Example: Simple Architecture

```mermaid
block-beta
    columns 3

    Browser["Browser Client"]:3

    space
    LB["Load Balancer"]
    space

    API1["API Server 1"]
    API2["API Server 2"]
    API3["API Server 3"]

    space
    DB[("Database")]
    Cache[("Redis Cache")]

    Browser --> LB
    LB --> API1
    LB --> API2
    LB --> API3
    API1 --> DB
    API2 --> DB
    API3 --> DB
    API1 --> Cache
```

### Example: CI/CD Pipeline

```mermaid
block-beta
    columns 5

    Source["Source Code"]:1
    Build["Build"]:1
    Test["Test"]:1
    Stage["Staging"]:1
    Prod["Production"]:1

    Source --> Build --> Test --> Stage --> Prod
```

---

## 🔹 Sankey Diagram

Sankey diagrams show **flows and their quantities** between nodes. The width of each link is proportional to the flow quantity. Ideal for visualizing energy flows, budget transfers, traffic sources, and resource allocation.

**Declaration keyword:** `sankey-beta`

> **Note:** This diagram type is currently in beta. Syntax may change in future Mermaid versions.

### Syntax

The data format is **CSV-like** with three columns per row:

| Column | Meaning |
|---|---|
| Source | The origin node name |
| Target | The destination node name |
| Value | The flow quantity (numeric) |

Each row is a comma-separated triple: `Source,Target,Value`. Nodes are created automatically from the source and target names.

### Key Points

- No explicit node declaration needed -- nodes are inferred from the data rows.
- Link width scales proportionally to the value.
- The same node name can appear as both a source and a target, creating multi-hop flows.
- Nodes with the same name are treated as the same node.

### Example: Energy Flow

```mermaid
sankey-beta

Electricity,Residential,30
Electricity,Commercial,25
Electricity,Industrial,45
Natural Gas,Residential,20
Natural Gas,Commercial,15
Natural Gas,Industrial,35
Renewable,Electricity,40
Fossil Fuel,Electricity,60
Fossil Fuel,Natural Gas,70
```

### Example: Website Traffic Flow

```mermaid
sankey-beta

Google,Landing Page,500
Social Media,Landing Page,200
Direct,Landing Page,150
Referral,Landing Page,100
Landing Page,Sign Up,300
Landing Page,Product Page,450
Landing Page,Bounce,200
Product Page,Purchase,120
Product Page,Cart Abandon,180
Sign Up,Active User,200
Sign Up,Churned,100
```

---

**See also:** [[Syntax Basics]] | [[Flowcharts]] | [[Styling and Themes]]