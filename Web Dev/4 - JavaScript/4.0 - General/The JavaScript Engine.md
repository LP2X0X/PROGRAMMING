---
tags:
  - js
  - term
  - general
---

- Today, JavaScript can execute not only in the browser, but also on the server, or actually on any device that has a special program called [the JavaScript engine](https://en.wikipedia.org/wiki/JavaScript_engine).
- The browser has an embedded engine sometimes called a “JavaScript virtual machine”.
- Different engines have different “codenames”. For example:

| Engine | Used by |
|---|---|
| [V8](https://en.wikipedia.org/wiki/V8_\(JavaScript_engine\)) | Chrome, Edge, Node.js, Deno |
| [SpiderMonkey](https://en.wikipedia.org/wiki/SpiderMonkey) | Firefox |
| JavaScriptCore (Nitro) | Safari |
| Chakra | IE (legacy) |

## Why It's Called a “Virtual Machine”

- It works similarly to how the **CLR** works for .NET: it takes higher-level code and runs it on an abstract machine rather than compiling directly to native CPU instructions ahead of time.
- Like the CLR, it manages memory (GC), handles execution, and JIT-compiles hot paths.

## How It Works (Simplified Pipeline)

1. **Parsing** — the engine reads JS source and builds an Abstract Syntax Tree (AST)
2. **Bytecode generation** — the AST is compiled into bytecode (an intermediate representation), similar to how C# compiles to IL
3. **Interpretation + JIT compilation** — the bytecode is initially interpreted, but “hot” paths (frequently executed code) get compiled to optimized native machine code at runtime (Just-In-Time compilation)

## JS Engine vs .NET CLR

| | JS Engine | .NET CLR |
|---|---|---|
| **Input** | Raw source code (no separate compile step) | Pre-compiled IL from the C# compiler |
| **Memory management** | GC (automatic) | GC (automatic) |
| **JIT compilation** | Yes (hot paths) | Yes (all IL on first call) |
| **Type system** | Dynamic | Static |