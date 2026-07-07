---
tags:
  - uml
  - deployment-diagram
  - structural
---

# Deployment Diagram

## 🔹 What It Shows

A **Deployment Diagram** models the physical deployment of **software artifacts** onto **hardware and infrastructure nodes**. It answers one fundamental question: **"What runs where?"**

It maps the runtime architecture of a system — which executables, libraries, and configuration files land on which servers, containers, or devices, and how those nodes communicate.

Use deployment diagrams when:
- Documenting infrastructure for a system (on-prem or cloud)
- Planning DevOps pipelines, CI/CD targets, and container orchestration
- Communicating architecture to system administrators and operations teams
- Reviewing deployment topology in architecture meetings
- Modeling network protocols and communication paths between machines
- Showing how [[Component Diagram]] elements map to physical hardware

```ad-info
Deployment diagrams are the **only** UML diagram type that deals with hardware topology. Every other diagram is either logical or behavioral.
```

---

## 🔹 Quick Reference

| Element                    | Notation                          | Description                                                    |
| -------------------------- | --------------------------------- | -------------------------------------------------------------- |
| **Node**                   | 3D box                            | Computational resource (hardware or software environment)      |
| **Device**                 | `<<device>>` stereotype on node   | Physical hardware: server, PC, router, mobile device           |
| **Execution Environment**  | `<<executionEnvironment>>` on node| Software platform: JVM, Docker, IIS, .NET CLR                  |
| **Artifact**               | `<<artifact>>` rectangle          | Deployable file: DLL, JAR, EXE, config, SQL script             |
| **Communication Path**     | Solid line between nodes          | Network or inter-process connection                            |
| **Protocol Label**         | `<<HTTP>>`, `<<TCP/IP>>`, etc.    | Technology/protocol on a communication path                    |
| **Deployment Relationship**| Artifact nested inside a node     | Shows that the artifact is deployed on that node               |
| **Dependency**             | Dashed arrow                      | One artifact depends on another                                |
| **Deployment Spec**        | `<<deploymentSpec>>` rectangle    | Configuration for a deployed artifact (tagged values)          |

---

## 🔹 Node Notation

A **node** represents any computational resource that can host artifacts. Nodes are drawn as **3D boxes** (a box with a raised top-left perspective edge) to distinguish them from regular class/component rectangles.

### Device Node `<<device>>`

A **device** is a physical piece of hardware.

```
    ┌──────────────────────────────┐
   ╱                              ╱│
  ╱──────────────────────────────╱ │
  │                              │ │
  │       <<device>>             │ │
  │       WebServer01            │ │
  │                              │ │
  │   OS: Windows Server 2022   │ │
  │   CPU: 8 cores              │ ╱
  │                              │╱
  └──────────────────────────────┘
```

### Execution Environment Node `<<executionEnvironment>>`

An **execution environment** is a software platform that hosts artifacts at runtime — it is not a physical machine but a process or runtime that runs on one.

```
    ┌──────────────────────────────┐
   ╱                              ╱│
  ╱──────────────────────────────╱ │
  │                              │ │
  │  <<executionEnvironment>>    │ │
  │  .NET CLR 8.0                │ │
  │                              │ ╱
  │                              │╱
  └──────────────────────────────┘
```

```ad-tip
Use `<<device>>` for things you can physically touch (server rack, laptop, router).
Use `<<executionEnvironment>>` for things you install or configure (Docker, IIS, JVM, Node.js runtime).
```

---

## 🔹 Artifacts

An **artifact** is a physical file that gets deployed — the tangible output of your build process.

### Notation

Artifacts use a rectangle with the `<<artifact>>` stereotype:

```
  ┌─────────────────────────┐
  │ <<artifact>>        ◇   │
  │ MyApp.dll                │
  └─────────────────────────┘
```

Common artifact types:
- `.dll`, `.exe` — .NET assemblies
- `.jar`, `.war` — Java archives
- `.js`, `.css`, `.html` — web assets
- `appsettings.json`, `web.config` — configuration files
- `Dockerfile`, `docker-compose.yml` — container definitions
- `.sql` — database migration scripts
- Docker images (e.g., `myapp:latest`)

### Artifact Deployed Inside a Node

When an artifact is drawn **inside** a node, it means that artifact is deployed on that node:

```
    ┌────────────────────────────────────┐
   ╱                                    ╱│
  ╱────────────────────────────────────╱ │
  │  <<device>>                        │ │
  │  AppServer                         │ │
  │                                    │ │
  │   ┌──────────────────────────┐     │ │
  │   │ <<artifact>>         ◇  │     │ │
  │   │ WebApp.dll               │     │ │
  │   └──────────────────────────┘     │ │
  │                                    │ │
  │   ┌──────────────────────────┐     │ │
  │   │ <<artifact>>         ◇  │     │ │
  │   │ appsettings.json         │     │ ╱
  │   └──────────────────────────┘     │╱
  └────────────────────────────────────┘
```

### Deployment Specification

A **deployment specification** provides configuration details for a deployed artifact using tagged values:

```
  ┌─────────────────────────────┐
  │ <<deploymentSpec>>          │
  │ web.config                  │
  ├─────────────────────────────┤
  │ maxConnections = 100        │
  │ timeout = 30s               │
  │ environment = "Production"  │
  └─────────────────────────────┘
```

---

## 🔹 Communication Paths

A **communication path** is a solid line between two nodes showing that they can exchange data. Label each path with the protocol or technology used.

```
  ┌──────────────┐                            ┌──────────────┐
 ╱              ╱│        <<HTTPS>>           ╱              ╱│
╱──────────────╱ │  ────────────────────     ╱──────────────╱ │
│  <<device>>  │ │                          │  <<device>>  │ │
│  Client PC   │ │                          │  WebServer   │ │
│              │ ╱                          │              │ ╱
└──────────────┘╱                           └──────────────┘╱
```

### Protocol Stereotypes

| Stereotype     | Usage                                   |
| -------------- | --------------------------------------- |
| `<<HTTP>>`     | Web traffic (unencrypted)               |
| `<<HTTPS>>`    | Encrypted web traffic                   |
| `<<TCP/IP>>`   | Raw TCP connections                     |
| `<<gRPC>>`     | Google RPC framework                    |
| `<<JDBC>>`     | Java database connectivity              |
| `<<ODBC>>`     | Open database connectivity              |
| `<<AMQP>>`     | Message queue protocol (RabbitMQ)       |
| `<<WebSocket>>`| Persistent bidirectional connections    |

### Multiplicity

Communication paths can show **multiplicity** just like associations in [[Class Diagram]]:

```
  ┌──────────┐          <<TCP/IP>>          ┌──────────┐
  │ AppServer│ ──────────────────────────── │ DBServer │
  │          │ *                        1   │          │
  └──────────┘                              └──────────┘

  (Many app servers connect to one database server)
```

---

## 🔹 Nested Nodes

Execution environments are typically **nested inside** device nodes to show the software stack running on the hardware. This is one of the most common and useful patterns in deployment diagrams.

### Basic Nesting

```
    ┌──────────────────────────────────────────────┐
   ╱                                              ╱│
  ╱──────────────────────────────────────────────╱ │
  │  <<device>>                                  │ │
  │  Linux Server                                │ │
  │                                              │ │
  │   ┌────────────────────────────────────┐     │ │
  │   │  <<executionEnvironment>>          │     │ │
  │   │  Docker Container                  │     │ │
  │   │                                    │     │ │
  │   │   ┌──────────────────────────┐     │     │ │
  │   │   │ <<artifact>>         ◇  │     │     │ │
  │   │   │ api-service:latest       │     │     │ │
  │   │   └──────────────────────────┘     │     │ │
  │   │                                    │     │ │
  │   └────────────────────────────────────┘     │ │
  │                                              │ ╱
  │                                              │╱
  └──────────────────────────────────────────────┘
```

### Multiple Execution Environments on One Device

```
    ┌──────────────────────────────────────────────────────────┐
   ╱                                                          ╱│
  ╱──────────────────────────────────────────────────────────╱ │
  │  <<device>>                                              │ │
  │  Windows Server                                          │ │
  │                                                          │ │
  │   ┌────────────────────┐     ┌────────────────────┐      │ │
  │   │ <<execEnvironment>>│     │ <<execEnvironment>>│      │ │
  │   │ IIS 10.0           │     │ SQL Server 2022    │      │ │
  │   │                    │     │                    │      │ │
  │   │ ┌────────────────┐ │     │ ┌────────────────┐ │      │ │
  │   │ │ <<artifact>> ◇ │ │     │ │ <<artifact>> ◇ │ │      │ │
  │   │ │ MyWebApp.dll   │ │     │ │ AppDB.mdf      │ │      │ │
  │   │ └────────────────┘ │     │ └────────────────┘ │      │ ╱
  │   └────────────────────┘     └────────────────────┘      │╱
  └──────────────────────────────────────────────────────────┘
```

---

## 🔹 Real-World Examples

### Example 1: Simple Web Application Deployment

A three-tier web app: browser, web server, database server.

```
  ┌─────────────────────┐       <<HTTPS>>       ┌───────────────────────────────────┐
 ╱                     ╱│                       ╱                                   ╱│
╱─────────────────────╱ │  ────────────────    ╱───────────────────────────────────╱ │
│  <<device>>         │ │                      │  <<device>>                       │ │
│  Client Machine     │ │                      │  Web Server                       │ │
│                     │ │                      │                                   │ │
│  ┌────────────────┐ │ │                      │  ┌─────────────────────────────┐  │ │
│  │ <<execEnv>>    │ │ │                      │  │ <<executionEnvironment>>    │  │ │
│  │ Web Browser    │ │ │                      │  │ IIS / Nginx                 │  │ │
│  │                │ │ │                      │  │                             │  │ │
│  │ ┌────────────┐ │ │ │                      │  │  ┌───────────────────────┐  │  │ │
│  │ │<<artifact>>│ │ │ │                      │  │  │ <<artifact>>      ◇  │  │  │ │
│  │ │index.html  │ │ │ │                      │  │  │ MyWebApp.dll          │  │  │ │
│  │ │app.js      │ │ │ │                      │  │  └───────────────────────┘  │  │ │
│  │ │styles.css  │ │ │ │                      │  │                             │  │ │
│  │ └────────────┘ │ │ │                      │  │  ┌───────────────────────┐  │  │ │
│  └────────────────┘ │ │                      │  │  │ <<artifact>>      ◇  │  │  │ │
│                     │ ╱                      │  │  │ appsettings.json      │  │  │ │
│                     │╱                       │  │  └───────────────────────┘  │  │ │
└─────────────────────┘                        │  └─────────────────────────────┘  │ │
                                               │                                   │ ╱
                                               │                                   │╱
                                               └───────────────────────────────────┘
                                                                 │
                                                                 │ <<TCP/IP>>
                                                                 │ port 3306
                                                                 │
                                               ┌───────────────────────────────────┐
                                              ╱                                   ╱│
                                             ╱───────────────────────────────────╱ │
                                             │  <<device>>                       │ │
                                             │  Database Server                  │ │
                                             │                                   │ │
                                             │  ┌─────────────────────────────┐  │ │
                                             │  │ <<executionEnvironment>>    │  │ │
                                             │  │ MariaDB 11.x               │  │ │
                                             │  │                             │  │ │
                                             │  │  ┌───────────────────────┐  │  │ │
                                             │  │  │ <<artifact>>      ◇  │  │  │ │
                                             │  │  │ app_schema.sql        │  │  │ │
                                             │  │  └───────────────────────┘  │  │ │
                                             │  │                             │  │ │
                                             │  │  ┌───────────────────────┐  │  │ │
                                             │  │  │ <<artifact>>      ◇  │  │  │ │
                                             │  │  │ app_data.ibd          │  │  │ │
                                             │  │  └───────────────────────┘  │  │ │
                                             │  └─────────────────────────────┘  │ │
                                             │                                   │ ╱
                                             │                                   │╱
                                             └───────────────────────────────────┘
```

### Example 2: Cloud Deployment with Load Balancer

A horizontally scaled cloud deployment with containerized services and a database cluster.

```
                            ┌────────────────────────┐
                           ╱                        ╱│
                          ╱────────────────────────╱ │
                          │  <<device>>            │ │
                          │  Load Balancer         │ │
                          │  (AWS ALB / Azure LB)  │ ╱
                          │                        │╱
                          └────────────────────────┘
                              │              │
                   <<HTTPS>>  │              │  <<HTTPS>>
                              │              │
          ┌───────────────────┘              └───────────────────┐
          │                                                      │
          ▼                                                      ▼
┌──────────────────────────────────┐    ┌──────────────────────────────────┐
│  <<device>>                      │    │  <<device>>                      │
│  App Server 1                    │    │  App Server 2                    │
│                                  │    │                                  │
│  ┌────────────────────────────┐  │    │  ┌────────────────────────────┐  │
│  │ <<executionEnvironment>>  │  │    │  │ <<executionEnvironment>>  │  │
│  │ Docker Container          │  │    │  │ Docker Container          │  │
│  │                            │  │    │  │                            │  │
│  │ ┌──────────────────────┐  │  │    │  │ ┌──────────────────────┐  │  │
│  │ │ <<artifact>>      ◇ │  │  │    │  │ │ <<artifact>>      ◇ │  │  │
│  │ │ api-service:2.1      │  │  │    │  │ │ api-service:2.1      │  │  │
│  │ └──────────────────────┘  │  │    │  │ └──────────────────────┘  │  │
│  │ ┌──────────────────────┐  │  │    │  │ ┌──────────────────────┐  │  │
│  │ │ <<artifact>>      ◇ │  │  │    │  │ │ <<artifact>>      ◇ │  │  │
│  │ │ config.yaml          │  │  │    │  │ │ config.yaml          │  │  │
│  │ └──────────────────────┘  │  │    │  │ └──────────────────────┘  │  │
│  └────────────────────────────┘  │    │  └────────────────────────────┘  │
│                                  │    │                                  │
└──────────────────────────────────┘    └──────────────────────────────────┘
          │                                                      │
          │                  <<TCP/IP>>                           │
          │                  port 3306                            │
          └──────────────────────┬───────────────────────────────┘
                                 │
                                 ▼
┌────────────────────────────────────────────────────────────────────────┐
│  <<device>>                                                            │
│  Database Cluster                                                      │
│                                                                        │
│  ┌──────────────────────────────┐  ┌──────────────────────────────┐    │
│  │ <<executionEnvironment>>    │  │ <<executionEnvironment>>    │    │
│  │ MariaDB Primary             │  │ MariaDB Replica             │    │
│  │                              │  │                              │    │
│  │ ┌────────────────────────┐  │  │ ┌────────────────────────┐  │    │
│  │ │ <<artifact>>        ◇ │  │  │ │ <<artifact>>        ◇ │  │    │
│  │ │ app_data.ibd           │──┼─▶│ │ app_data.ibd           │  │    │
│  │ └────────────────────────┘  │  │ └────────────────────────┘  │    │
│  │                              │  │                (read-only)  │    │
│  └──────────────────────────────┘  └──────────────────────────────┘    │
│                                       replication ──────────▶         │
└────────────────────────────────────────────────────────────────────────┘
```

### Example 3: Desktop Application (.NET Developer Workstation)

A C#/.NET desktop application connecting to a remote database — the kind of setup a desktop developer works with daily.

```
  ┌──────────────────────────────────────────────────────┐
 ╱                                                      ╱│
╱──────────────────────────────────────────────────────╱ │
│  <<device>>                                          │ │
│  Developer Workstation                               │ │
│  (Windows 11, 32 GB RAM)                             │ │
│                                                      │ │
│  ┌────────────────────────────────────────────────┐  │ │
│  │ <<executionEnvironment>>                      │  │ │
│  │ .NET Runtime 8.0                              │  │ │
│  │                                                │  │ │
│  │  ┌──────────────────────┐                      │  │ │
│  │  │ <<artifact>>      ◇ │                      │  │ │
│  │  │ InspectionApp.exe    │                      │  │ │
│  │  └──────────────────────┘                      │  │ │
│  │                                                │  │ │
│  │  ┌──────────────────────┐                      │  │ │
│  │  │ <<artifact>>      ◇ │                      │  │ │
│  │  │ DataAccess.dll       │                      │  │ │
│  │  └──────────────────────┘                      │  │ │
│  │                                                │  │ │
│  │  ┌──────────────────────┐                      │  │ │
│  │  │ <<artifact>>      ◇ │                      │  │ │
│  │  │ appsettings.json     │                      │  │ │
│  │  └──────────────────────┘                      │  │ │
│  │                                                │  │ │
│  └────────────────────────────────────────────────┘  │ │
│                                                      │ │
│  ┌────────────────────────────────────────────────┐  │ │
│  │ <<executionEnvironment>>                      │  │ │
│  │ Visual Studio 2022                            │  │ │
│  │                                                │  │ │
│  │  ┌──────────────────────┐                      │  │ │
│  │  │ <<artifact>>      ◇ │                      │  │ │
│  │  │ InspectionApp.sln    │                      │  │ ╱
│  └────────────────────────────────────────────────┘  │╱
└──────────────────────────────────────────────────────┘
                        │
                        │ <<TCP/IP>>
                        │ port 3306
                        │
                        ▼
        ┌──────────────────────────────────────┐
       ╱                                      ╱│
      ╱──────────────────────────────────────╱ │
      │  <<device>>                          │ │
      │  Database Server (Remote)            │ │
      │                                      │ │
      │  ┌────────────────────────────────┐  │ │
      │  │ <<executionEnvironment>>      │  │ │
      │  │ MariaDB 11.4                  │  │ │
      │  │                                │  │ │
      │  │  ┌──────────────────────────┐  │  │ │
      │  │  │ <<artifact>>          ◇ │  │  │ │
      │  │  │ inspection_db            │  │  │ │
      │  │  └──────────────────────────┘  │  │ │
      │  │                                │  │ │
      │  │  ┌──────────────────────────┐  │  │ │
      │  │  │ <<artifact>>          ◇ │  │  │ │
      │  │  │ migrations_v3.sql        │  │  │ ╱
      │  │  └──────────────────────────┘  │  │╱
      │  └────────────────────────────────┘  │
      └──────────────────────────────────────┘
```

```ad-info
In a real project, the deployment diagram for a desktop app is simpler than for a web app — but it still clarifies which DLLs ship to the user's machine versus what stays on the server.
```

---

## 🔹 Deployment vs Component Diagram

| Aspect               | [[Component Diagram]]                                | Deployment Diagram                            |
| --------------------- | ---------------------------------------------------- | --------------------------------------------- |
| **Focus**             | Logical architecture                                 | Physical architecture                         |
| **Shows**             | Components, interfaces, dependencies                 | Nodes, artifacts, communication paths         |
| **Answers**           | "What are the parts and how do they plug together?"   | "What runs where and how do they connect?"     |
| **Hardware**          | Not shown                                            | Explicitly modeled                            |
| **Protocols**         | Not specified                                        | Named on communication paths                  |
| **Typical audience**  | Developers, architects                               | DevOps, sysadmins, architects                 |

In practice, these two diagrams are **complementary**. You design components first (logical), then map them to deployment nodes (physical). A component like `OrderService` becomes an artifact `OrderService.dll` deployed on a specific server node.

```ad-tip
A useful workflow: draw a [[Component Diagram]] for the logical design, then create a deployment diagram that shows where each component's artifact lands. Cross-reference the two in architecture documentation.
```

---

## 🔹 Tips

1. **Start with nodes, not artifacts.** Sketch out the hardware/infrastructure topology first (what servers, containers, and environments exist), then place artifacts into them. This keeps the diagram organized.

2. **Don't model every file.** Show only the significant, deployable artifacts — the main EXE/DLL, key config files, and database assets. Omit individual `.cs` source files and transient build outputs.

3. **Use nested nodes generously.** The pattern `<<device>>` containing `<<executionEnvironment>>` containing `<<artifact>>` is the bread and butter of deployment diagrams. It makes the software stack explicit.

4. **Always label communication paths.** A line between two nodes is meaningless without a protocol label. `<<HTTPS>>` vs `<<gRPC>>` vs `<<TCP/IP>>` tells the reader how those nodes talk — which matters for firewall rules, security, and debugging.

5. **Mirror your real environment names.** Use actual server names, container image tags, and port numbers when possible. `AppServer-Prod-01` is more useful than `Server A` in an architecture review.

6. **Keep one diagram per deployment scope.** A single diagram for the entire enterprise becomes unreadable. Instead, create separate deployment diagrams per subsystem, environment (dev/staging/prod), or tier.

```ad-warning
Deployment diagrams become outdated fast. Treat them as living documents — update them when infrastructure changes, not just when code changes.
```

---

See also: [[Component Diagram]], [[Package Diagram]], [[Class Diagram]]
