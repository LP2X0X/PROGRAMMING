---
tags:
  - uml
  - basics
---

## 🔹 What Is UML?

**UML** (Unified Modeling Language) is a standardized visual modeling language maintained by the **Object Management Group (OMG)**. It provides a set of graphical notations for creating abstract models of software systems — their structure, behavior, and architecture.

UML is *not* a programming language or a methodology. It is a **specification language** — a way to describe a system so that developers, architects, and stakeholders can communicate about it unambiguously.

> [!info] Key Point
> UML is a **communication tool first**. Its value is making complex systems understandable through diagrams. A quick whiteboard sketch using UML notation is still "UML."

---

## 🔹 Brief History

| Year | Event |
|------|-------|
| Late 1980s | Dozens of competing OO modeling methods emerge (the "method wars") |
| 1994 | **Grady Booch** (Booch method), **James Rumbaugh** (OMT), **Ivar Jacobson** (OOSE) join forces at Rational Software |
| 1996 | UML 0.9 draft circulated |
| 1997 | **UML 1.0** submitted to OMG and adopted as a standard |
| 2005 | **UML 2.0** — major revision adding sequence diagram fragments, composite structures, interaction overview diagrams |
| 2017 | **UML 2.5.1** — current version, streamlined and clarified specification |

The "**Three Amigos**" (Booch, Rumbaugh, Jacobson) unified their individual OO modeling methodologies into a single standard, ending the fragmentation that had plagued the industry.

---

## 🔹 Why Learn UML?

- **Communication** — universal vocabulary for discussing system design with other developers and architects
- **Documentation** — diagrams convey structure faster than paragraphs of prose
- **Design thinking** — drawing before coding forces you to think through relationships and responsibilities
- **Interviews** — system design questions at senior-level roles often expect UML-style diagrams on a whiteboard
- **Industry standard** — widely used in enterprise, embedded, and safety-critical software (medical devices, aviation)
- **Pattern literacy** — GoF design patterns are almost always illustrated with UML class diagrams
- **Code generation** — some tools (Enterprise Architect, MagicDraw) can generate skeleton code from UML models

---

## 🔹 The 14 Diagram Types

UML 2.5 defines **14 diagram types** grouped into two categories:

### Structural Diagrams (7)

Show the **static structure** of the system — what exists and how it is organized.

| Diagram | Purpose | Frequency |
|---------|---------|-----------|
| [[Class Diagram]] | Classes, attributes, operations, and relationships between them | **Common** |
| [[Object Diagram]] | Snapshot of object instances and their values at a moment in time | Occasional |
| [[Component Diagram]] | High-level software components and their provided/required interfaces | Occasional |
| [[Composite Structure Diagram]] | Internal structure of a class or component (parts, ports, connectors) | Rare |
| Package Diagram | Grouping of classes/elements into packages and their dependencies | Occasional |
| Deployment Diagram | Physical deployment of software artifacts to hardware nodes | Occasional |
| Profile Diagram | Extension of UML metamodel with custom stereotypes | Rare |

### Behavioral Diagrams (7)

Show what the system **does** — how it behaves and reacts over time.

| Diagram | Purpose | Frequency |
|---------|---------|-----------|
| Use Case Diagram | Actors and system functionality from the user's perspective | **Common** |
| Activity Diagram | Workflows, business processes, algorithms (like a flowchart, but richer) | **Common** |
| State Machine Diagram | States and transitions of a single object throughout its lifecycle | Occasional |
| Sequence Diagram | Object interactions over time — who sends what message to whom, in order | **Common** |
| Communication Diagram | Object interactions emphasizing structural links (was "Collaboration Diagram") | Rare |
| Timing Diagram | State changes of objects along an explicit time axis | Rare |
| Interaction Overview Diagram | High-level flow combining activity and interaction diagrams | Rare |

> [!tip] The Big Four
> In practice, you will use **class diagrams**, **sequence diagrams**, **activity diagrams**, and **use case diagrams** roughly 80% of the time. Master those first and add others as needed.

### Diagram Taxonomy (Tree View)

```
UML Diagrams
├── Structural ("what exists")
│   ├── Class Diagram          ← foundation of OO modeling
│   ├── Object Diagram
│   ├── Component Diagram
│   ├── Composite Structure Diagram
│   ├── Package Diagram
│   ├── Deployment Diagram
│   └── Profile Diagram
└── Behavioral ("what happens")
    ├── Use Case Diagram
    ├── Activity Diagram
    ├── State Machine Diagram
    └── Interaction Diagrams
        ├── Sequence Diagram   ← most-used behavioral diagram
        ├── Communication Diagram
        ├── Timing Diagram
        └── Interaction Overview Diagram
```

---

## 🔹 When to Use UML (and When It's Overkill)

**Use UML when:**
- Communicating a design to a team — even informal whiteboard sketches
- Documenting a complex domain model so new team members can ramp up
- Designing system architecture *before* implementation begins
- Discussing design patterns (Observer, Strategy, Factory, etc.)
- Writing technical proposals, RFCs, or architecture decision records
- Working on safety-critical or regulated software (often legally required)
- Onboarding developers to a large legacy codebase

**Skip UML when:**
- The code is simple and self-documenting — a 50-line script needs no diagrams
- You are prototyping or spiking — the code *is* the design, and it will change rapidly
- Your team does not read UML — use whatever notation your team understands
- You would spend more time on the diagram than on the code itself
- The system is already well-documented with inline comments and clean architecture

> [!warning] Common Misconception
> UML does *not* require exhaustive, tool-generated diagrams with every attribute and method modeled. "Just enough UML" — a quick whiteboard sketch showing key classes and relationships — is perfectly valid and is how most experienced developers actually use it.

---

## 🔹 UML vs Other Notations

| Notation | Best For | Relationship to UML |
|----------|----------|---------------------|
| **ERD** (Entity-Relationship Diagram) | Database schema design, table relationships | Class diagram covers similar ground but is OOP-focused, not table-focused |
| **Flowchart** | Simple sequential processes | Activity diagram is UML's richer equivalent (supports parallelism, swim lanes) |
| **BPMN** (Business Process Model & Notation) | Business process modeling | Overlaps with activity diagrams but adds business-specific semantics (events, gateways) |
| **ArchiMate** | Enterprise architecture (strategy → technology) | Higher abstraction level than UML; models business and technology layers together |
| **C4 Model** | Software architecture at multiple zoom levels | Complements UML; intentionally simpler notation (Context, Container, Component, Code) |
| **Mermaid / PlantUML** | Diagrams as code | Text-based renderers for UML-like diagrams; great for version-controlled docs |

> [!tip]
> UML and these notations are **not mutually exclusive**. Many teams use C4 for high-level architecture views and UML class/sequence diagrams for detailed design.

---

## 🔹 Structural vs Behavioral — Mental Model

Think of it this way:

- **Structural diagrams** answer: *"What are the building blocks, and how are they connected?"*
  - Classes, objects, components, packages, deployment nodes
  - Analogous to a **building's blueprint** — rooms, walls, wiring

- **Behavioral diagrams** answer: *"What happens when the system runs?"*
  - Use cases, activities, state changes, message sequences
  - Analogous to a **building's evacuation plan** — who goes where, when, in what order

Most real-world modeling uses a mix: a [[Class Diagram]] to define structure, then [[Sequence Diagram]]s to show how those classes interact at runtime.

---

## 🔹 Tips: "Just Enough UML"

1. **Start with class diagrams** — they map most directly to code (especially in C#, Java, C++)
2. **Draw before you code** — even 5 minutes of sketching prevents hours of rework
3. **Keep it informal** — whiteboard UML > no UML at all. Do not let tool friction stop you
4. **Don't model everything** — focus on the parts that are hard to understand from code alone (complex relationships, tricky state machines, non-obvious workflows)
5. **Iterate** — your first diagram will be wrong. That is the point — it is cheaper to refactor a diagram than to refactor code
6. **Version your diagrams** — use text-based tools (PlantUML, Mermaid) so diagrams live in source control alongside the code
7. **Know your audience** — use simpler diagrams for stakeholders, detailed ones for developers
8. **Use consistent notation** — even informal diagrams should follow the standard symbols (see [[Common Notation]])

> [!example] Quick Sketch vs Full Model
> **Quick sketch** (5 min, whiteboard): 4–6 key classes, their relationships, maybe key methods. Perfect for a design discussion.
>
> **Full model** (hours, tool-generated): every class, attribute, method, multiplicity, constraint. Needed for code generation or regulatory compliance, but overkill for most projects.

See also: [[Common Notation]], [[Relationships]], [[Class Diagram]]
