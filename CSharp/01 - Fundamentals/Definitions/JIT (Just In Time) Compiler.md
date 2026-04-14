---
tags: 
 - csharp
 - definition
---

## Definition

The JIT compiler is a component of the CLR (Common Language Runtime) that converts **IL (Intermediate Language) code** into **native machine code** at runtime, right before execution. It compiles methods on demand — only when they are called for the first time — and caches the compiled native code so subsequent calls run directly without recompilation.

## Why It Is Needed

1. **Platform independence** — C# compiles to IL, not to machine code. IL is platform-agnostic, but the CPU cannot execute IL directly. The JIT compiler bridges this gap by translating IL into native instructions specific to the machine it is running on.

2. **Runtime optimization** — Because compilation happens at runtime, the JIT compiler can apply optimizations based on the actual hardware (CPU architecture, cache sizes, instruction sets like AVX/SSE) rather than relying on generic ahead-of-time assumptions.

3. **Security and verification** — Before compiling, the JIT compiler performs type-safety and memory-safety verification on the IL, ensuring that the code does not perform illegal operations.

## How It Works

```
C# Source Code
      |  (Roslyn compiler)
      v
  IL / MSIL  (stored in .dll / .exe assemblies)
      |  (at runtime, when a method is first called)
      v
  JIT Compiler (part of CLR)
      |
      v
  Native Machine Code  (cached for reuse)
```
