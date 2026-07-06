---
tags:
 - csharp
 - attributes
---

## Built-in Attributes

.NET ships with many predefined attributes. These are consumed by the compiler, runtime, serializers, or framework tools -- you don't need to reflect on them yourself.

## Compiler Attributes

These affect compilation behavior directly.

### `[Obsolete]`

Marks code as deprecated. The compiler emits a warning (or error) when it's used:

```csharp
[Obsolete("Use NewMethod() instead")]
public void OldMethod() { }

[Obsolete("Removed in v3", error: true)]  // compile ERROR, not just a warning
public void ReallyOldMethod() { }
```

### `[Conditional]`

Method is only *called* if a compilation symbol is defined. The method itself still exists in IL, but **call sites are stripped out**:

```csharp
[Conditional("DEBUG")]
public static void DebugLog(string msg)
{
    Console.WriteLine(msg);
}

// In Release builds (no DEBUG symbol), calls to DebugLog() are removed
// entirely by the compiler -- the method body still exists but is never invoked
```

```ad-warning
`[Conditional]` only works on methods that return `void`. The compiler can't strip a call site that is expected to produce a return value.
```

### `[CallerMemberName]`, `[CallerFilePath]`, `[CallerLineNumber]`

The compiler injects caller info at the call site. Useful for logging and `INotifyPropertyChanged`:

```csharp
public void Log(string msg,
    [CallerMemberName] string member = "",
    [CallerFilePath] string file = "",
    [CallerLineNumber] int line = 0)
{
    Console.WriteLine($"{file}:{line} [{member}] {msg}");
}

// Call site -- you don't pass the extra args, the compiler fills them in
Log("something happened");
```

## Runtime / Serialization Attributes

### `[Serializable]`

Marks a class as serializable for binary/SOAP serialization (.NET Framework). Less relevant in modern .NET where JSON/XML serializers use their own attributes (`[JsonProperty]`, `[XmlElement]`, etc.).

### `[NonSerialized]`

Excludes a **field** from serialization:

```csharp
[Serializable]
public class User
{
    public string Name;
    [NonSerialized] public string TempCache;  // skipped during serialization
}
```

### `[ThreadStatic]`

Makes a static field per-thread (each thread gets its own copy):

```csharp
[ThreadStatic]
static int _perThreadCounter;  // each thread starts with default(int) = 0
```

```ad-warning
`[ThreadStatic]` does not run the field initializer on secondary threads. If you write `[ThreadStatic] static int _x = 42;`, only the main thread sees `42` -- other threads see `0`. Use `ThreadLocal<T>` if you need initialized per-thread values.
```

## Interop Attributes

### `[DllImport]`

Declares a method implemented in an unmanaged DLL (P/Invoke):

```csharp
[DllImport("user32.dll")]
public static extern int MessageBox(IntPtr hWnd, string text, string caption, uint type);
```

### `[StructLayout]` and `[MarshalAs]`

Control memory layout for interop with native code. `[StructLayout(LayoutKind.Explicit)]` lets you specify exact byte offsets with `[FieldOffset]`.

## Common Attribute Summary

| Attribute | Purpose | Consumer |
|---|---|---|
| `[Obsolete]` | Mark deprecated code | Compiler |
| `[Conditional]` | Strip calls unless symbol defined | Compiler |
| `[Serializable]` | Enable binary serialization | BinaryFormatter |
| `[DllImport]` | Call unmanaged DLL functions | CLR P/Invoke |
| `[ThreadStatic]` | Per-thread static field | CLR |
| `[Flags]` | Bitwise enum support | `Enum.ToString()`, `HasFlag` |
| `[CallerMemberName]` | Inject caller info | Compiler |
| `[InternalsVisibleTo]` | Expose internals to another assembly | Compiler |

## See Also

- [[Attribute Overview]]
- [[Custom Attributes]]
- [[Assembly-Level Attributes Overview]]
