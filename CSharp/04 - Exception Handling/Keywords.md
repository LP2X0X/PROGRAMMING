---
tags:
 - csharp
 - exception-handling
---

# Exception Handling Keywords

## Three Categories of Errors

- **Bugs** — mistakes made by the programmer. Example: forgetting to dispose a resource, off-by-one errors, null dereferences. These should be fixed in code, not caught at runtime.
- **User errors** — caused by the person running the app. Example: entering letters in a numeric field, providing a malformed file path. Handle these with input validation and clear error messages.
- **Exceptions** — runtime anomalies that are difficult to predict or prevent. Example: database goes offline, file is corrupted, network is unreachable. These are what the `try/catch` mechanism is designed for.


---

## `try` / `catch`

Wrap code that might throw in `try`, and handle the exception in `catch`. You can extract details from the caught exception object.

```csharp
try
{
    File.ReadAllText("data.txt");
}
catch (FileNotFoundException ex)
{
    Console.WriteLine($"File not found: {ex.FileName}");
}
catch (Exception ex)
{
    Console.WriteLine($"Something went wrong: {ex.Message}");
}
```

- Catch **specific** exceptions first, then more general ones.
- The `Exception` base class catch-all should come last (if used at all).


---

## `finally`

Code inside `finally` **always runs** — whether an exception was thrown or not. Use it for cleanup: closing files, disposing objects, releasing connections.

```csharp
FileStream? fs = null;
try
{
    fs = File.OpenRead("data.txt");
    // work with file
}
catch (IOException ex)
{
    Console.WriteLine(ex.Message);
}
finally
{
    fs?.Dispose(); // always runs
}
```

In practice, a `using` statement often replaces `try/finally` for disposable objects:

```csharp
using var fs = File.OpenRead("data.txt");
// fs.Dispose() is called automatically when scope ends
```


---

## `throw`

Re-throws an exception to the caller, or raises a new one.

```csharp
// Throw a new exception
if (age < 0)
    throw new ArgumentOutOfRangeException(nameof(age), "Age cannot be negative.");

// Re-throw the current exception (preserves stack trace)
catch (Exception ex)
{
    Log(ex);
    throw; // NOT throw ex — that resets the stack trace
}
```

- **`throw;`** — preserves the original stack trace (preferred for re-throwing).
- **`throw ex;`** — resets the stack trace to this point (loses where the error actually originated).
