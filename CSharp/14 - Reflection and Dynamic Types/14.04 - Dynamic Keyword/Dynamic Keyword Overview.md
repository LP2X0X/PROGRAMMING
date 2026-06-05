---
tags:
 - csharp
 - dynamic
---

## What Is the `dynamic` Keyword?

`dynamic` tells the compiler: "don't check anything on this variable — resolve it all at runtime." No compile-time type checking, no IntelliSense, no safety net. The type, members, and operations are all discovered when the code actually runs.

```csharp
dynamic x = "Hello";
Console.WriteLine(x.Length);   // works — string has Length

x = 42;
Console.WriteLine(x.Length);   // compiles fine, but CRASHES at runtime
                                // int doesn't have Length
```

The compiler emits no error on the second call because it skips all checks on `dynamic`. The error only surfaces when the CLR tries to resolve `.Length` on an `int` at runtime.


---

## How `dynamic` Works Under the Hood

When the compiler sees `dynamic`, it doesn't just throw away type info — it emits code that uses the **Dynamic Language Runtime (DLR)**. At each call site, the DLR:

1. Inspects the actual runtime type of the object
2. Looks up the member (method, property, field)
3. Caches the result for future calls with the same type (**call site caching**)
4. Invokes the member

```
COMPILE TIME                                  RUNTIME

dynamic obj = GetSomething();                 DLR resolves:
obj.DoWork();                                 1. What is obj's actual type?
                                              2. Does it have DoWork()?
Compiler emits a "call site"                  3. Cache the binding
instead of a direct method call               4. Invoke DoWork()
```

Under the hood, `dynamic` is actually `object` in the IL — with special attributes that tell the DLR to handle member resolution:

```csharp
// What you write:
dynamic x = 10;

// What the IL stores:
[Dynamic] object x = 10;
```


---

## `dynamic` vs `var` vs `object`

```csharp
var x = "Hello";       // compiler knows it's string — full IntelliSense, compile-time checks
object y = "Hello";    // compiler knows it's object — must cast to access string members
dynamic z = "Hello";   // compiler knows nothing — all checks deferred to runtime
```

| | `var` | `object` | `dynamic` |
|---|---|---|---|
| Type resolved at | Compile time | Compile time | Runtime |
| Type safety | Full | Must cast | None until runtime |
| IntelliSense | Full | Only object members | None |
| Can change type? | No | Yes (but cast needed) | Yes (freely) |
| Boxing value types? | No | Yes | Yes (hidden) |
| Performance | Best | Cast overhead | DLR overhead |

See [[Difference between var, object and dynamic keywords]] for detailed examples.


---

## When to Use `dynamic`

### 1. Simplifying Reflection / Late Binding

Without `dynamic` — verbose reflection code:

```csharp
Type type = assembly.GetType("CarLibrary.MiniVan");
object obj = Activator.CreateInstance(type);
MethodInfo mi = type.GetMethod("TurboBoost");
mi.Invoke(obj, null);
```

With `dynamic` — the DLR handles the lookup:

```csharp
Type type = assembly.GetType("CarLibrary.MiniVan");
dynamic obj = Activator.CreateInstance(type);
obj.TurboBoost();   // DLR resolves this at runtime
```

Same result, far less boilerplate. See [[Simplifying Late-Bound Calls Using Dynamic Types]] for the full example.

### 2. COM Interop

COM objects often don't have compile-time type info. `dynamic` eliminates the ugly casting:

```csharp
// Without dynamic — lots of casting and object[]
object excel = Activator.CreateInstance(Type.GetTypeFromProgID("Excel.Application"));
excel.GetType().InvokeMember("Visible", BindingFlags.SetProperty, null, excel, new object[] { true });

// With dynamic — reads like normal code
dynamic excel = Activator.CreateInstance(Type.GetTypeFromProgID("Excel.Application"));
excel.Visible = true;
excel.Workbooks.Add();
excel.Cells[1, 1].Value = "Hello from C#";
```

### 3. Interop with Dynamic Languages

When calling code from IronPython, IronRuby, or JavaScript engines running on the [[Dynamic Language Runtime|DLR]], `dynamic` lets you interact naturally without knowing the types.


---

## What `dynamic` Is NOT

```ad-warning
`dynamic` does NOT make C# a dynamically typed language. It's a **local escape hatch** from compile-time checking. The rest of your code is still statically typed. Use it only when you genuinely need runtime resolution — not as a shortcut to avoid thinking about types.
```

Common misconceptions:

| Misconception | Reality |
|---|---|
| "`dynamic` creates a new thread" | No — it has nothing to do with threading |
| "`dynamic` is faster than reflection" | Same mechanism, slightly more overhead (DLR call sites). But the code is much cleaner |
| "`dynamic` is the same as `var`" | `var` is compile-time inference. `dynamic` is runtime resolution. Completely different |
| "`dynamic` is the same as `object`" | `object` requires explicit casting. `dynamic` resolves members automatically at runtime |


---

## ExpandoObject — Dynamic Object Creation

`ExpandoObject` lets you create objects with members added at runtime:

```csharp
dynamic person = new ExpandoObject();
person.Name = "Long";
person.Age = 28;
person.Greet = new Action(() => Console.WriteLine($"Hi, I'm {person.Name}"));

person.Greet();   // "Hi, I'm Long"

// Add new members at any time
person.Email = "long@example.com";
```

Under the hood, `ExpandoObject` implements `IDictionary<string, object>`, so each "property" is really a key-value pair. This is useful for scenarios like JSON deserialization where the shape isn't known ahead of time.


---

## Limitations and Costs

### Performance

`dynamic` is slower than direct calls because the DLR must resolve types, cache bindings, and handle dispatch at every call site. For hot paths, this matters.

### No IntelliSense

The IDE can't help you — no autocomplete, no parameter hints, no "go to definition." You're flying blind.

### No LINQ Support

```csharp
dynamic list = new List<int> { 1, 2, 3 };
var result = list.Where(x => x > 1);   // compile error — lambdas can't bind to dynamic
```

LINQ depends on compile-time type info that `dynamic` doesn't provide. Cast to the concrete type first.

### Runtime Errors Instead of Compile Errors

Every mistake that static typing would catch at build time becomes a runtime `RuntimeBinderException`. Missing method? Runtime crash. Wrong argument type? Runtime crash.

### Refactoring Breaks

Rename a method that `dynamic` calls? The compiler won't catch the stale call site. It compiles fine and crashes at runtime.

See [[Limitation]] for the full list.


---

## When to Avoid `dynamic`

- You know the type — use it directly
- You can define an interface — [[Building extendable application|plugin pattern]] is safer
- You're inside a hot loop — the DLR overhead adds up
- You want IDE support — IntelliSense doesn't work with `dynamic`

```ad-note
If you find yourself using `dynamic` frequently, reconsider your design. Most C# applications should have zero or near-zero uses of `dynamic`. The main legitimate cases are COM interop, simplifying one-off reflection, and interacting with dynamic language runtimes.
```


---

## See Also

- [[Difference between var, object and dynamic keywords]] — detailed comparison
- [[Dynamic Language Runtime]] — the infrastructure behind `dynamic`
- [[Expression Trees]] — how the DLR represents code as data
- [[Simplifying Late-Bound Calls Using Dynamic Types]] — reflection vs `dynamic`
- [[Limitation]] — full list of `dynamic` limitations
- [[Early Binding and Late Binding]] — the binding concepts behind `dynamic`
