---
tags:
 - csharp
 - exception-handling
---

## `Exception.StackTrace`

The `StackTrace` property is a string that captures the call chain from where the exception was thrown back to where it was caught. It is set automatically when the exception is created -- you never assign it yourself.

### Basic Usage

```csharp
try
{
    MethodA();
}
catch (Exception ex)
{
    Console.WriteLine(ex.StackTrace);
}

// Output (example):
//    at MyApp.Service.MethodC() in Service.cs:line 42
//    at MyApp.Service.MethodB() in Service.cs:line 28
//    at MyApp.Program.MethodA() in Program.cs:line 15
```

### `throw` vs `throw ex` -- Critical Difference

```csharp
// CORRECT - preserves the original stack trace
catch (Exception ex)
{
    LogError(ex);
    throw;          // stack trace points to the original throw site
}

// WRONG - resets the stack trace to this line
catch (Exception ex)
{
    LogError(ex);
    throw ex;       // stack trace now starts HERE, original context is lost
}
```

### Re-throwing with Full Trace via `ExceptionDispatchInfo`

When you need to store an exception and re-throw it later (e.g., across async boundaries), use `ExceptionDispatchInfo` to preserve the full trace:

```csharp
using System.Runtime.ExceptionServices;

ExceptionDispatchInfo? captured = null;

try
{
    DoWork();
}
catch (Exception ex)
{
    captured = ExceptionDispatchInfo.Capture(ex);
}

// Later...
captured?.Throw(); // full original stack trace is preserved
```

### Tips & Best Practices

- **Never use `throw ex;` in a catch block.** This is the single most common exception handling mistake in C#. It destroys the original stack trace and makes debugging significantly harder.
- **`StackTrace` is only populated after the exception is thrown.** If you create an exception with `new Exception()` but never throw it, `StackTrace` is `null`.
- **Line numbers require PDB/debug symbols.** In release builds without symbols, you'll see method names but not line numbers. Ship PDBs (or use embedded PDBs) for production if you want full traces in your logs.
- **Don't parse `StackTrace` as a string.** If you need programmatic access to stack frames, use `System.Diagnostics.StackTrace`:
  ```csharp
  var trace = new System.Diagnostics.StackTrace(ex, fNeedFileInfo: true);
  StackFrame frame = trace.GetFrame(0);
  Console.WriteLine($"File: {frame.GetFileName()}, Line: {frame.GetFileLineNumber()}");
  ```
- **`InnerException` chains have their own traces.** When wrapping exceptions, each level preserves its own `StackTrace`. Use `ex.ToString()` to get the full chain including all inner stack traces.
- **Async methods produce noisy traces.** `async`/`await` introduces state machine frames (like `MoveNext()`) in the stack trace. The `[StackTraceHidden]` attribute (C# 10+) and `ExceptionDispatchInfo` help clean this up.
- **Don't log `StackTrace` to end users.** It exposes internal implementation details (method names, file paths, line numbers). Log it server-side; show users a correlation ID instead.
