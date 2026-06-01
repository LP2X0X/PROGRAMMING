---
tags:
 - csharp
 - processes
---

## What Is a Process?

A process is an **operating system-level concept** that describes a running program. More formally, it is a container for:

1. **Memory space** — a private, isolated block of virtual memory where the application's data lives
2. **Executable code** — the compiled instructions that make up the program
3. **External code libraries** — DLLs and shared libraries the program loads
4. **At least one thread** — the primary thread that serves as the entry point (see [[The Role of Threads]])
5. **A set of OS resources** — handles to files, network sockets, registry keys, etc.

```ad-note
For each .NET Core application loaded into memory, the OS creates a separate and isolated process for use during its lifetime.
```


---

## Process Isolation

Each process runs in its **own isolated memory space**. This has two important consequences:

1. **Stability** — if one process crashes, it does not affect other running processes. The OS tears down that process's memory without corrupting others.
2. **Security** — data in one process cannot be directly read or written by another process.

This makes a process a **fixed, safe boundary** for a running application.

If you *do* need processes to communicate, you must use explicit inter-process communication (IPC) mechanisms:

| IPC Mechanism | Namespace / Class | Use Case |
|---|---|---|
| Named Pipes | `System.IO.Pipes` | Streaming data between processes on the same machine |
| Memory-Mapped Files | `System.IO.MemoryMappedFiles.MemoryMappedFile` | Sharing a region of memory between processes |
| Sockets / TCP | `System.Net.Sockets` | Communication between processes on same or different machines |


---

## Process Identification (PID)

```ad-note
Every Windows process is assigned a unique **process identifier (PID)** and may be independently loaded and unloaded by the OS as necessary (as well as programmatically).
```

You can see PIDs in Task Manager, or query them programmatically:

```csharp
using System.Diagnostics;

// Get the current process
Process current = Process.GetCurrentProcess();
Console.WriteLine($"PID: {current.Id}");
Console.WriteLine($"Name: {current.ProcessName}");
Console.WriteLine($"Started: {current.StartTime}");

// List all running processes
foreach (Process proc in Process.GetProcesses())
{
    Console.WriteLine($"  {proc.Id,-8} {proc.ProcessName}");
}
```


---

## What a Process Contains — Visual Overview

```
┌─────────────────────────────────────────┐
│              Process (PID 1234)          │
│                                         │
│  ┌─────────────┐   ┌────────────────┐   │
│  │ Primary      │   │ Worker         │   │
│  │ Thread       │   │ Thread(s)      │   │
│  └─────────────┘   └────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │ Virtual Memory Space            │    │
│  │  - Stack (per thread)           │    │
│  │  - Heap (shared within process) │    │
│  │  - Static/global data           │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │ Loaded Modules (DLLs)           │    │
│  │  - CoreCLR runtime              │    │
│  │  - Application assemblies       │    │
│  │  - Third-party libraries        │    │
│  └─────────────────────────────────┘    │
│                                         │
│  ┌─────────────────────────────────┐    │
│  │ OS Resources (handles)          │    │
│  │  - Files, sockets, registry     │    │
│  └─────────────────────────────────┘    │
└─────────────────────────────────────────┘
```


---

## Process vs. Thread vs. AppDomain

| Concept                             | What it is                                  | Isolation level                                                         |
| ----------------------------------- | ------------------------------------------- | ----------------------------------------------------------------------- |
| **Process**                         | A running program with its own memory space | Full — separate memory, separate PID                                    |
| **Thread**                          | A path of execution within a process        | None — threads share the process's memory                               | 
| **AppDomain** (.NET Framework only) | A logical subdivision within a process      | Partial — isolated within the same process (not available in .NET Core) |

In .NET Core / .NET 5+, AppDomains are gone. Process is the isolation boundary, and threads are the concurrency mechanism within it.
