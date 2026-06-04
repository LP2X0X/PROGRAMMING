---
tags:
 - csharp
 - reflection
 - metadata
---

## The User Strings Metadata Token

Every string literal in your source code is stored in the **User Strings** metadata stream (also called the `#US` heap). The CLR references these strings by token at runtime rather than embedding them inline in the IL instructions.

### How String Literals Appear in Metadata

```csharp
public class Greeter
{
    public void SayHello()
    {
        Console.WriteLine("Hello, World!");
        Console.WriteLine("Welcome to .NET");
    }
}
```

In the metadata User Strings heap:

```
User Strings
-----------------------------------------------
70000001 : (16) L"Hello, World!"
70000021 : (15) L"Welcome to .NET"
```

Each entry has a **token** (the hex number starting with `70`) and the string content. The `L` prefix indicates a UTF-16 encoded string, which is .NET's internal string representation.

### How IL Code References User Strings

The IL instruction `ldstr` pushes a string from the User Strings heap onto the evaluation stack by token:

```il
IL_0001: ldstr      "Hello, World!"    // loads from User Strings token 70000001
IL_0006: call       void [System.Console]System.Console::WriteLine(string)
IL_000b: ldstr      "Welcome to .NET"  // loads from User Strings token 70000021
IL_0010: call       void [System.Console]System.Console::WriteLine(string)
```

The JIT compiler resolves the token to the actual string object at runtime.

```ad-important
All string literals are clearly visible in the assembly metadata. This has serious security implications — **never store passwords, API keys, connection strings, or other sensitive data as string literals** in your code. Anyone with access to the assembly can read them using `ildasm.exe`, `dotnet-dump`, or even a hex editor. Use secure storage mechanisms like environment variables, Azure Key Vault, or the .NET Secret Manager instead.
```

### String Interning

The CLR **interns** string literals by default. When the runtime encounters a `ldstr` instruction, it checks whether an identical string already exists in the intern pool:

- If yes, it returns a reference to the existing string (no new allocation)
- If no, it creates the string, adds it to the intern pool, and returns it

This means identical string literals across your assembly share the same runtime object, saving memory. However, it also means the strings persist for the lifetime of the application — they are never garbage collected.

```ad-note
String interning applies to **literals** only. Strings built at runtime (via concatenation, `StringBuilder`, etc.) are *not* automatically interned unless you explicitly call `string.Intern()`.
```

### What Doesn't Appear in User Strings

Not all strings in your program end up in the User Strings heap:

| String Source | In User Strings? | Why |
|---|---|---|
| String literals (`"hello"`) | Yes | Compiler emits them directly |
| Interpolated string constants | Yes | Compiler resolves them at compile time |
| Runtime-built strings | No | Created on the managed heap at runtime |
| `nameof()` expressions | Yes | Compiler resolves to a string literal |
| Resource strings (`.resx`) | No | Stored in embedded resources, not the US heap |

## See Also

- [[TypeDef and TypeRef]]
- [[Field]]
- [[How to generate the IL file of an assembly]]
