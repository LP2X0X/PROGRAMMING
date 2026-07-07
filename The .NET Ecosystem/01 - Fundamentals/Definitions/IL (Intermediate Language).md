---
tags: 
 - csharp
 - definition
---

## Definition

**IL (Intermediate Language)**, also known as **MSIL (Microsoft Intermediate Language)** or **CIL (Common Intermediate Language)**, is the low-level, CPU-independent instruction set that all .NET languages compile down to. It sits between your high-level source code (C#, F#, VB.NET) and the native machine code that the CPU actually executes.

IL is not source code and it is not machine code — it is a **platform-neutral binary format** stored inside .NET assemblies (`.dll` / `.exe` files).

## Why It Is Needed

1. **Language interoperability** — C#, F#, and VB.NET all compile to the same IL. This means a class written in F# can be consumed by C# seamlessly, because at the IL level they speak the same language.

2. **Platform independence** — IL is not tied to any specific CPU architecture. The same assembly can run on x86, x64, or ARM as long as a compatible .NET runtime is installed to [[JIT (Just In Time) Compiler|JIT compile]] it.

3. **Security and verification** — The CLR can inspect and verify IL before executing it, enforcing type safety and memory safety at load time.

4. **Optimization at runtime** — Because IL defers machine code generation to the [[JIT (Just In Time) Compiler]], the runtime can apply hardware-specific optimizations that a compile-time compiler cannot predict.

## Usage

### Viewing IL

You can inspect the IL of any .NET assembly using:

- **`ildasm`** (IL Disassembler) — ships with the .NET SDK
- **ILSpy** or **dnSpy** — popular open-source decompilers
- **`dotnet-ildasm`** — cross-platform .NET global tool

```bash
# Install the global tool
dotnet tool install -g dotnet-ildasm

# Disassemble an assembly
dotnet ildasm MyApp.dll -o MyApp.il
```

### Example

A simple C# method:

```csharp
public static int Add(int a, int b)
{
    return a + b;
}
```

Compiles to the following IL:

```
.method public hidebysig static int32 Add(int32 a, int32 b) cil managed
{
    .maxstack 2
    ldarg.0      // push 'a' onto the stack
    ldarg.1      // push 'b' onto the stack
    add          // pop both, push their sum
    ret          // return the value on top of the stack
}
```

### Key Characteristics

- **Stack-based** — IL uses an evaluation stack rather than registers. Operands are pushed onto the stack, operations consume them and push results back.
- **Strongly typed** — Every IL instruction knows the types it operates on.
- **Object-oriented** — IL natively supports classes, interfaces, inheritance, and virtual dispatch.
