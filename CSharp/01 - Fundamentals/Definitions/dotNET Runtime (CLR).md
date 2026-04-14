---
tags: 
 - csharp
 - definition
---

## Definition

The .NET runtime, also called the **CLR (Common Language Runtime)**, is the managed execution engine that runs .NET applications. It sits between compiled IL code and the operating system, providing essential services like memory management, JIT compilation, and type safety.

## Core Responsibilities

1. **JIT compilation** — converts [[JIT (Just In Time) Compiler|IL to native machine code]] at runtime
2. **Memory management** — allocates and frees memory automatically via the **Garbage Collector (GC)**
3. **Type safety** — enforces type checking and memory safety at runtime
4. **Exception handling** — provides a structured exception model across all .NET languages
5. **Thread management** — manages the thread pool and synchronization primitives
6. **Security** — enforces code access security and IL verification

## How It Fits Together

```
C# Source Code
     |  (compile time - Roslyn)
     v
IL Code in assemblies (.dll)
     |  (runtime)
     v
.NET Runtime (CLR)
  ├── JIT Compiler       → native code
  ├── Garbage Collector   → memory
  ├── Type System         → safety
  └── Thread Pool         → concurrency
     |
     v
Operating System / Hardware
```
