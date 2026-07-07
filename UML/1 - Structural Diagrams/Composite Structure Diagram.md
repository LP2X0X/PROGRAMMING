---
tags:
  - uml
  - composite-structure-diagram
  - structural
---

## 🔹 What It Shows

A Composite Structure Diagram reveals the **internal structure of a classifier** (class, component, or collaboration) **at runtime**. Unlike a [[Class Diagram]] that shows static relationships between types, this diagram shows how instances inside a containing classifier are **wired together** to fulfill its behavior.

Think of it as opening up a class and looking at the parts inside -- what objects it holds, how they connect to each other, and how the outside world reaches them through ports.

> This is one of the **rarest UML diagrams** in practice. Most teams never draw one. But when you need to show the internal wiring of a complex component, nothing else captures it as precisely.

## 🔹 Core Elements

### Parts

A **part** is a runtime instance (or set of instances) owned by the containing classifier. It represents a role that an internal object plays inside the whole.

- Drawn as a rectangle inside the parent classifier
- Syntax: `partName : TypeName [multiplicity]`
- A part is NOT the same as an attribute -- it emphasizes the role and connectivity, not just data

### Ports

A **port** is an interaction point on the boundary of a classifier. It defines a distinct point through which the classifier communicates with its environment or routes messages to internal parts.

- Drawn as a **small square on the edge** of the classifier boundary
- Can be **typed** by an interface (provided or required)
- Provided interface: the port offers this contract (lollipop `--O`)
- Required interface: the port needs this contract (socket `--C`)
- A port can be **behavioral** (handled by the classifier itself) or **delegated** (forwarded to an internal part via a connector)

### Connectors

A **connector** is a link between two parts, or between a port and a part. It shows that instances communicate at runtime.

- **Assembly connector**: links a provided interface to a required interface between two parts (ball-and-socket)
- **Delegation connector**: links a port on the boundary to an internal part, forwarding external requests inward

### Collaborations

A **collaboration** defines a set of cooperating roles and their connectors to accomplish a specific goal. It is drawn as a **dashed ellipse** containing the role names. Collaboration uses (dashed ellipse with a dashed arrow to the classifier) show how a collaboration pattern maps onto actual parts.

## 🔹 Notation Examples

### A Classifier with Internal Parts and Ports

```
+--------------------------------------------------+
|                  <<component>>                    |
|                    Server                         |
|                                                   |
|  +-----------------+      +-------------------+   |
|  | pool :          |      | handler :         |   |
|  | ThreadPool [1]  |----->| RequestHandler [1]|   |
|  +-----------------+      +-------------------+   |
|                                  |                |
|                                  |                |
|                           +------v--------+      |
|                           | log :         |      |
|                           | Logger [1]    |      |
|                           +---------------+      |
|                                                   |
+--[port]-------------------------------------------+
   [http]
```

- `Server` is the containing classifier
- `pool`, `handler`, and `log` are **parts** (internal instances)
- `[http]` is a **port** on the boundary
- Lines between parts are **connectors**

### Ports with Provided/Required Interfaces

```
                        +-----------------------------+
                        |         WebServer           |
                        |                             |
    provided            |   +-------------------+     |
    interface           |   | handler :         |     |
        O--[httpPort]------>| RequestHandler    |     |
                        |   +-------------------+     |
                        |           |                 |
    required            |           v                 |
    interface           |   +-------------------+     |
        C--[dbPort]-----<---| repo :            |     |
                        |   | Repository        |     |
                        |   +-------------------+     |
                        +-----------------------------+

    O--  = provided interface (lollipop)
    C--  = required interface (socket)
    ---> = delegation connector (port to internal part)
```

- `httpPort` provides an interface -- external clients connect here
- `dbPort` requires an interface -- this component needs a database
- The arrows from ports to parts are **delegation connectors**

### Assembly Connector Between Two Components

```
+------------------+                  +------------------+
|   OrderService   |                  |  PaymentService  |
|                  |                  |                  |
|             [pay]|--O----C--[order]|                  |
|                  |  provided  req  |                  |
+------------------+                  +------------------+

    O----C  = assembly connector (ball-and-socket)
```

## 🔹 Real-World Example: Server Class

A `Server` class that processes HTTP requests. Internally it holds a thread pool, a request handler, and a logger. External traffic arrives through an HTTP port and is delegated to the handler.

```
+------------------------------------------------------------------+
|                        <<component>>                              |
|                          Server                                   |
|                                                                   |
|   +------------------+    dispatch    +---------------------+     |
|   | pool :           |-------------->| handler :           |     |
|   | ThreadPool [4]   |               | RequestHandler [1]  |     |
|   +------------------+               +---------------------+     |
|                                             |                    |
|                                             | log(msg)           |
|                                             v                    |
|                                      +--------------+            |
|                                      | log :        |            |
|              delegation              | Logger [1]   |            |
|   O--[httpPort]--------------------->+--------------+            |
|      IHttpListener                                               |
|                                                                   |
+------------------------------------------------------------------+

  Parts:
    pool     : ThreadPool [4]        -- 4 worker threads
    handler  : RequestHandler [1]    -- dispatches requests
    log      : Logger [1]            -- centralized logging

  Port:
    httpPort : IHttpListener         -- provided interface

  Connectors:
    httpPort ---delegation---> handler   (external requests forwarded)
    pool     ---assembly--->   handler   (threads dispatch to handler)
    handler  ---assembly--->   log       (handler writes to logger)
```

Reading this diagram tells you: external HTTP traffic enters through `httpPort`, gets delegated to `handler`, which is fed work by `pool` (4 threads), and writes diagnostic output to `log`.

## 🔹 This Diagram Is Rare in Practice

Most teams will **never** need a Composite Structure Diagram. Here is why:

- The internal wiring of a class is usually simple enough to understand from source code
- [[Component Diagram]]s cover most "what's inside this system" needs at a coarser grain
- Tooling support is weaker than for class or sequence diagrams
- Many developers have never encountered this diagram type

**Don't reach for this diagram by default.** Reach for it only when the situations below apply.

## 🔹 When It IS Useful

| Situation | Why Composite Structure Helps |
|---|---|
| Complex component with multiple interacting internal objects | Shows the runtime wiring that source code scatters across constructors and DI configs |
| Port-based architecture (SOA, hardware interfaces) | Ports and delegation connectors map directly to the design |
| Collaboration patterns (e.g., Observer, Mediator) | Collaboration uses show how a pattern maps onto real parts |
| Embedded / real-time systems | Internal structure and communication paths matter for timing and safety analysis |
| Explaining a framework's internal architecture to newcomers | A single diagram replaces pages of prose about what holds what |

When the goal is just "what components exist and how they depend on each other" at a system level, prefer a [[Component Diagram]] instead.

## 🔹 Composite Structure vs Component Diagram

| Aspect | Composite Structure Diagram | [[Component Diagram]] |
|---|---|---|
| **Focus** | Internal runtime structure of ONE classifier | Dependencies between MANY components |
| **Granularity** | Fine -- parts, ports, connectors inside a class/component | Coarse -- components, interfaces, dependencies |
| **Shows runtime instances?** | Yes (parts with multiplicities) | No (types and dependencies only) |
| **Ports** | Central element | Supported but less emphasized |
| **Connectors** | Assembly + Delegation connectors between parts | Dependency arrows between components |
| **Typical use** | "How does this component work internally?" | "What are the major pieces of the system?" |
| **Frequency in practice** | Rare | Common |
| **Best for** | Detailed internal wiring of a complex class | High-level system architecture overview |

**Rule of thumb:** Use a [[Component Diagram]] to show the *outside view* (what components exist and their interfaces). Use a Composite Structure Diagram to show the *inside view* (how one component is wired internally).

## 🔹 Quick Notation Reference

| Element | Notation | Description |
|---|---|---|
| Classifier | Large rectangle with name | The containing class or component |
| Part | `name : Type [mult]` rectangle inside classifier | Runtime instance playing a role |
| Port | Small square on classifier boundary | Interaction point for external communication |
| Provided interface | Lollipop `--O` on port | Interface the port offers |
| Required interface | Socket `--C` on port | Interface the port needs |
| Assembly connector | Ball-and-socket `O----C` | Links provided to required interface between parts |
| Delegation connector | Line from port to internal part | Forwards external messages to a part |
| Collaboration | Dashed ellipse with role names | A pattern of cooperating roles |
| Collaboration use | Dashed ellipse + dashed arrow to classifier | Maps collaboration roles onto actual parts |
| Multiplicity | `[n]` or `[n..m]` after type | Number of instances in that role |
| Stereotype | `<<component>>`, `<<service>>` | Classifier kind (uses [[Common Notation]]) |

Connectors follow the same general principles as [[Relationships]] but are scoped to runtime links within a single classifier rather than design-time associations between types.

See also: [[Component Diagram]], [[Common Notation]], [[Relationships]]
