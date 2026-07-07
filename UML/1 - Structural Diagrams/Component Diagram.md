---
tags:
  - uml
  - component-diagram
  - structural
---

## 🔹 What It Shows

A component diagram models the **high-level physical structure** of a system in terms of its software components, their interfaces, and the dependencies between them. It answers: "What are the major replaceable parts of this system, and how do they plug together?"

Components are **modular, deployable, and replaceable** units. Unlike [[Class Diagram]]s that show internal class structure, component diagrams operate at a coarser granularity -- think DLLs, NuGet packages, services, or entire subsystems.

Use component diagrams when you need to:
- Communicate architecture to stakeholders who don't need class-level detail
- Plan which teams own which parts of the system
- Show how third-party libraries and services integrate
- Document deployment boundaries and interface contracts

## 🔹 Component Notation

A component is drawn as a rectangle with the **component stereotype** `<<component>>` or the **component icon** -- a small rectangle with two protruding tabs on its left side:

```
 ┌───────────────────────────┐
 │  ┌──┐                     │
 │──┤  │   OrderService      │
 │  └──┘                     │
 │                           │
 └───────────────────────────┘

   Compact icon form:

 ┌──┐┌──────────────────┐
 │──┤│  OrderService     │
 │  └│                   │
 └───┘───────────────────┘

   Stereotype form:

 ┌───────────────────────────┐
 │    <<component>>          │
 │    OrderService           │
 └───────────────────────────┘
```

The icon form is the most common in practice. The stereotype keyword form works when you're sketching quickly. Both are valid UML.

## 🔹 Provided Interface (Lollipop / Ball)

A **provided interface** is a contract that a component implements and exposes to the outside world. It is drawn as a line ending in a small circle (the "lollipop" or "ball"):

```
 ┌───────────────────────┐
 │  ┌──┐                 │
 │──┤  │  OrderService   │──○  IOrderProcessing
 │  └──┘                 │
 └───────────────────────┘

   ──○   = "I provide this interface"
```

The lollipop says: "I fulfill this contract -- you can call me through it." This corresponds directly to a class implementing an interface (e.g., `OrderService : IOrderProcessing` in C#).

Multiple provided interfaces are drawn as separate lollipops:

```
 ┌───────────────────────┐
 │  ┌──┐                 │──○  IOrderProcessing
 │──┤  │  OrderService   │
 │  └──┘                 │──○  IOrderQuery
 └───────────────────────┘
```

## 🔹 Required Interface (Socket / Cup)

A **required interface** is a contract that a component needs someone else to fulfill. It is drawn as a line ending in a half-circle (the "socket" or "cup"):

```
                            ┌───────────────────────┐
                            │  ┌──┐                 │
 IPaymentGateway  ◠──       │──┤  │  OrderService   │
                            │  └──┘                 │
                            └───────────────────────┘

   ◠──   = "I require this interface"
```

The socket says: "I depend on this contract -- someone else must provide it for me to work." This maps to [[Dependency Inversion]] -- the component depends on an abstraction, not a concrete implementation.

## 🔹 Ball-and-Socket (Assembly Connector)

When a provided interface meets a required interface, they snap together in a **ball-and-socket** connector. This is the signature notation of the component diagram:

```
 ┌──────────────────┐           ┌──────────────────┐
 │  ┌──┐            │           │  ┌──┐            │
 │──┤  │  Payment   │──○──◠──  │──┤  │  Order     │
 │  └──┘  Service   │           │  └──┘  Service   │
 └──────────────────┘           └──────────────────┘
         provides                     requires
       IPaymentGateway            IPaymentGateway

   ──○──◠──   =  ball-and-socket / assembly connector
```

Read it as: PaymentService **provides** `IPaymentGateway`, and OrderService **requires** `IPaymentGateway`. The ball plugs into the socket.

This is the most important notation to master -- it shows how components are wired together through their contracts. The assembly connector replaces a plain [[Relationships|dependency arrow]] with a more expressive visual.

## 🔹 Ports

A **port** is a small square drawn on the boundary of a component. It represents a distinct interaction point -- a grouped set of provided and/or required interfaces accessible at that point:

```
 ┌──────────────────────────────┐
 │  ┌──┐                       │
 │──┤  │   WebServer           │
 │  └──┘                  ┌──┐ │
 │                        │  ├─│──○  IHTTP
 │                        └──┘ │
 │  ┌──┐                       │
 │  │  ├───────────────────────│──○  ILogging
 │  └──┘                       │
 └──────────────────────────────┘
      port                 port
```

Ports are useful when a component has **multiple distinct channels** of communication. For example, a web server might expose an HTTP port for clients and a management port for health checks, each with their own set of interfaces.

In practice, ports are used less often than bare lollipops. Use them when the grouping adds clarity.

## 🔹 Dependencies and Connectors

Beyond the ball-and-socket assembly connector, components can be connected with:

**Dependency arrow** (dashed arrow `---->`) -- a weaker, less specific connection. Use when you want to show "uses" without specifying a formal interface:

```
 ┌──────────────┐          ┌──────────────┐
 │  WebClient   │ -------> │  APIGateway  │
 │              │ <<use>>  │              │
 └──────────────┘          └──────────────┘
```

**Delegate connector** -- connects a port on the outer component to an inner component inside a composite (see [[Composite Structure Diagram]]):

```
 ┌─────────────────────────────────────┐
 │   <<component>>  AppServer          │
 │                                     │
 │  ┌──────────────┐                   │
 │  │  Router      │──○  IRouting      │
 │  └──────────────┘        │          │
 │                     delegate   ┌──┐ │
 │                          └─────┤  ├─│──○  IHTTP
 │                                └──┘ │
 └─────────────────────────────────────┘
```

See [[Relationships]] and [[Common Notation]] for the full set of arrow types.

## 🔹 Example: Web Application Architecture

```
 ┌────────────────┐
 │  ┌──┐          │
 │──┤  │ Browser  │
 │  └──┘ Client   │──○──◠── ┌────────────────┐
 └────────────────┘         │  ┌──┐          │
                     IHTTP  │──┤  │ Web API  │
                            │  └──┘ Server   │──○──◠── ┌────────────────┐
                            └────────────────┘         │  ┌──┐          │
                                          IDataAccess  │──┤  │ Database │
                            ┌────────────────┐         │  └──┘ Server   │
                            │  ┌──┐          │         └────────────────┘
                            │──┤  │ Auth     │──○──◠──┘
                            │  └──┘ Service  │   IAuth
                            └────────────────┘
```

Key takeaways:
- Browser communicates with Web API through `IHTTP`
- Web API depends on Database through `IDataAccess`
- Auth Service provides `IAuth`, consumed by Web API
- Each component is independently deployable

## 🔹 Example: Microservices Architecture

```
 ┌──────────────────────────────────────────────────────────┐
 │  <<component>>  API Gateway                              │
 └──────┬───────────────┬────────────────┬──────────────────┘
        │               │                │
   ──○──◠──        ──○──◠──         ──○──◠──
        │               │                │
 ┌──────┴──────┐  ┌─────┴───────┐  ┌────┴──────────┐
 │  ┌──┐       │  │  ┌──┐       │  │  ┌──┐         │
 │──┤  │ User  │  │──┤  │ Order │  │──┤  │ Inventory│
 │  └──┘ Svc   │  │  └──┘ Svc   │  │  └──┘ Svc     │
 └──────┬──────┘  └──────┬──────┘  └────┬───────────┘
        │                │               │
   ──○──◠──         ──○──◠──        ──○──◠──
        │                │               │
 ┌──────┴──────┐  ┌──────┴──────┐  ┌────┴───────────┐
 │  User DB    │  │  Order DB   │  │  Inventory DB  │
 └─────────────┘  └─────────────┘  └────────────────┘

 Each service owns its database (Database-per-Service pattern)
```

Each microservice is a component with well-defined interfaces. The API Gateway routes external requests to the appropriate service. Services communicate through their provided/required interfaces -- in practice, REST APIs, gRPC, or message queues.

## 🔹 Example: .NET Layered Application

A classic .NET solution structure mapped to a component diagram:

```
 ┌─────────────────────────┐
 │  ┌──┐                   │
 │──┤  │  Presentation     │    (ASP.NET MVC / Blazor / WPF)
 │  └──┘  MyApp.Web        │
 └────────────┬────────────┘
              │
         ──○──◠──  IBusinessServices
              │
 ┌────────────┴────────────┐
 │  ┌──┐                   │
 │──┤  │  Business Logic   │    (Class Library)
 │  └──┘  MyApp.Core       │
 └────────────┬────────────┘
              │
         ──○──◠──  IRepository
              │
 ┌────────────┴────────────┐
 │  ┌──┐                   │
 │──┤  │  Data Access      │    (EF Core / Dapper)
 │  └──┘  MyApp.Data       │
 └────────────┬────────────┘
              │
         ──○──◠──  ADO.NET / SQL
              │
 ┌────────────┴────────────┐
 │                         │
 │        Database         │    (SQL Server / MariaDB)
 │                         │
 └─────────────────────────┘
```

Each layer is a separate assembly (DLL). The interfaces between them (`IBusinessServices`, `IRepository`) enforce the dependency rule: upper layers depend on abstractions, not concrete lower layers. This is [[Dependency Inversion]] in action, typically wired up with DI in `Program.cs`.

## 🔹 Notation Quick-Reference

| Symbol | Name | Meaning |
|--------|------|---------|
| Rectangle + icon/stereotype | Component | A modular, replaceable unit of software |
| `──○` | Provided interface (lollipop) | "I implement this contract" |
| `◠──` | Required interface (socket) | "I need this contract fulfilled" |
| `──○──◠──` | Assembly connector (ball-and-socket) | Provided meets required |
| Small square on boundary | Port | Grouped interaction point |
| `----->` | Dependency | General "uses" relationship |
| `──────>` with `<<delegate>>` | Delegate connector | Port-to-inner-component link |
| `<<component>>` | Stereotype | Labels a rectangle as a component |
| Nesting | Subsystem | Components inside components |

## 🔹 When to Use vs. Other Diagrams

| Need | Diagram |
|------|---------|
| High-level parts and their interfaces | **Component Diagram** |
| Internal class structure | [[Class Diagram]] |
| Runtime instances and their links | [[Composite Structure Diagram]] |
| Physical deployment to servers/containers | [[Deployment Diagram]] |
| Module dependencies (packages) | [[Package Diagram]] |

See also: [[Common Notation]], [[Relationships]], [[Composite Structure Diagram]], [[Deployment Diagram]], [[Package Diagram]], [[Class Diagram]]
