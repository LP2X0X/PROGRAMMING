---
tags:
 - csharp
 - object-lifetime
 - disposable
---

## Why the `using` Keyword for Disposal?

When working with [[IDisposable]] types, you should call `Dispose()` when you're done. Wrapping every disposable in a `try`/`finally` is defensive but tedious. The `using` keyword is syntactic sugar that guarantees `Dispose()` is called, even if an exception is thrown.

```ad-warning
If you attempt to "use" an object that does not implement `IDisposable`, you will receive a compiler error.
```


---

## The `using` Statement (Block Syntax)

```csharp
using (var stream = new FileStream("data.txt", FileMode.Open))
{
    // use stream
}
// Dispose() called here — guaranteed, even on exception
```

The compiler transforms this into:

```csharp
FileStream stream = new FileStream("data.txt", FileMode.Open);
try
{
    // use stream
}
finally
{
    if (stream != null)
        ((IDisposable)stream).Dispose();
}
```

### Multiple Resources in One Block

You can declare multiple objects of the same type with a comma-delimited list:

```csharp
using (MyResource rw = new MyResource(), rw2 = new MyResource())
{
    // use rw and rw2
}
// both disposed
```

For different types, nest or use using declarations:

```csharp
using (var connection = new SqlConnection(connString))
using (var command = new SqlCommand(sql, connection))
{
    // use both
}
```


---

## Using Declarations (C# 8+)

A using declaration is a variable declaration preceded by the `using` keyword — no braces needed. The object is disposed at the end of the enclosing scope (typically the method):

```csharp
void ProcessFile()
{
    using var stream = new FileStream("data.txt", FileMode.Open);
    using var reader = new StreamReader(stream);

    string content = reader.ReadToEnd();
    // use content

}   // both reader and stream disposed here, in reverse order
```

### When to Use Which

| Syntax | Dispose happens at | Use when |
|---|---|---|
| `using (var x = ...) { }` | End of the block | You want explicit, narrow scope |
| `using var x = ...;` | End of the enclosing scope | Resource lives for the whole method |


---

## Disposal Order

When multiple `using` declarations are in the same scope, they dispose in **reverse order** — last created, first disposed (like a stack):

```csharp
using var connection = new SqlConnection(connString);
using var command = new SqlCommand(sql, connection);
using var reader = command.ExecuteReader();
// disposed: reader → command → connection
```

This reverse order matters — you can't close a connection before the command and reader using it are disposed.


---

## See Also

- [[IDisposable]] — the interface behind `using`
- [[Finalizer and Dispose]] — the full dispose pattern
- [[Disposable Object]] — disposable object overview
