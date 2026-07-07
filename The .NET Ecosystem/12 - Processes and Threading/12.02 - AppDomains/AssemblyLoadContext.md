---
tags:
 - csharp
 - appdomains
 - assemblies
---

# AssemblyLoadContext

## Where It Fits — The Full Hierarchy

In .NET, the isolation hierarchy is **Process → AppDomain → Load Contexts**:

```
┌─────────────────────────────────────────────────────────┐
│                     Windows Process                     │
│                                                         │
│  ┌───────────────────────────────────────────────────┐  │
│  │          Single AppDomain (.NET 5+)               │  │
│  │                                                   │  │
│  │  ┌─────────────────┐  ┌─────────────────┐        │  │
│  │  │  Default ALC    │  │  Custom ALC     │        │  │
│  │  │                 │  │  "Plugins"      │        │  │
│  │  │  MyApp.dll      │  │  Plugin.dll     │        │  │
│  │  │  System.dll     │  │  PluginDep.dll  │        │  │
│  │  └─────────────────┘  └─────────────────┘        │  │
│  │                                                   │  │
│  └───────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

In .NET 5+, there is always exactly **one AppDomain** per process — you cannot create more. But that single AppDomain can be further subdivided into multiple **load context boundaries**. A load context creates a scope for loading, resolving, and potentially unloading a set of assemblies. It provides a way for the single AppDomain to establish a "specific home" for a given object.

```ad-note
Load contexts handle only the **assembly loading/unloading** aspect that AppDomains used to provide. They do NOT provide the security sandboxing or full memory isolation that AppDomains did. For true isolation in .NET 5+, use separate processes. See [[AppDomain Overview]].
```


---

## The Default Context

Every .NET app starts with one ALC — `AssemblyLoadContext.Default`. When you write a normal `using ClassLibrary1;` or reference a NuGet package, those assemblies are loaded into the default context.

```csharp
var defaultCtx = AssemblyLoadContext.Default;
Console.WriteLine(defaultCtx.Name); // "Default"
```


---

## Loading the Same Assembly — Same Context vs Different Contexts

The **context boundary** is what creates isolation, not the number of load calls. These two examples show exactly why.

### Two Different Contexts — Full Isolation

Loading the same DLL into **two separate** contexts produces two completely independent copies:

```csharp
static void LoadInDifferentContexts()
{
    var path = Path.Combine(
        AppDomain.CurrentDomain.BaseDirectory,
        "ClassLibrary1.dll");

    AssemblyLoadContext lc1 = new AssemblyLoadContext("NewContext1", false);
    var cl1 = lc1.LoadFromAssemblyPath(path);
    var c1 = cl1.CreateInstance("ClassLibrary1.Car");

    AssemblyLoadContext lc2 = new AssemblyLoadContext("NewContext2", false);
    var cl2 = lc2.LoadFromAssemblyPath(path);
    var c2 = cl2.CreateInstance("ClassLibrary1.Car");

    Console.WriteLine($"Assembly1 Equals(Assembly2) {cl1.Equals(cl2)}");  // False
    Console.WriteLine($"Assembly1 == Assembly2 {cl1 == cl2}");            // False
    Console.WriteLine($"Class1.Equals(Class2) {c1.Equals(c2)}");         // False
    Console.WriteLine($"Class1 == Class2 {c1 == c2}");                   // False
}
```

**Everything is `False`** — different assembly objects, different types, different instances:

| Comparison | Result | Why |
|---|---|---|
| `cl1.Equals(cl2)` | `False` | Two different `Assembly` objects — loaded independently |
| `cl1 == cl2` | `False` | Different references, same physical file doesn't matter |
| `c1.Equals(c2)` | `False` | Two different objects on the heap |
| `c1 == c2` | `False` | Different objects, and their **types are also different** |

```ad-warning
The `Car` from `lc1` and the `Car` from `lc2` are **not the same type** — even though they came from the same source code and the same DLL file. You cannot cast one to the other. In .NET, a type's identity is **name + assembly + load context**. This is called **type identity by context**.
```

### One Context — Assembly is Deduplicated

Now the same experiment, but loading into **one** context twice:

```csharp
static void LoadInSameContext()
{
    var path = Path.Combine(
        AppDomain.CurrentDomain.BaseDirectory,
        "ClassLibrary1.dll");

    // ONE context, load the same DLL twice
    AssemblyLoadContext lc1 = new AssemblyLoadContext("NewContext1", false);
    var cl1 = lc1.LoadFromAssemblyPath(path);
    var c1 = cl1.CreateInstance("ClassLibrary1.Car");

    var cl2 = lc1.LoadFromAssemblyPath(path);  // same context!
    var c2 = cl2.CreateInstance("ClassLibrary1.Car");

    Console.WriteLine($"Assembly1 Equals(Assembly2) {cl1.Equals(cl2)}");  // True
    Console.WriteLine($"Assembly1 == Assembly2 {cl1 == cl2}");            // True
    Console.WriteLine($"Class1.Equals(Class2) {c1.Equals(c2)}");         // False
    Console.WriteLine($"Class1 == Class2 {c1 == c2}");                   // False
}
```

The **assemblies are the same** but the **objects are still different**:

| Comparison | Result | Why |
|---|---|---|
| `cl1.Equals(cl2)` | `True` | Context already loaded this DLL — `cl2` is just a pointer to the same assembly |
| `cl1 == cl2` | `True` | Same reference — the context doesn't load a second copy |
| `c1.Equals(c2)` | `False` | Two separate `CreateInstance` calls → two different objects on the heap |
| `c1 == c2` | `False` | Different object references (but they ARE the same **type** now) |

A load context **deduplicates** assemblies. When you call `LoadFromAssemblyPath` with a path it already loaded, it returns the existing `Assembly` reference. The DLL is only loaded once; the second call is a no-op that hands back the same object.

### Summary

```
Two contexts → two assemblies, two type systems, two sets of static fields
One context  → one assembly, one type system, shared static fields
```

The context boundary is what creates isolation. The number of load calls doesn't matter.

This is analogous to how multiple AppDomains worked in .NET Framework — each AppDomain had its own copy of static fields and types. In .NET 5+, multiple ALCs within the single AppDomain provide that same **type isolation** without needing separate AppDomains.


---

## The `isCollectible` Parameter

The second parameter in the `AssemblyLoadContext` constructor controls whether the context (and its assemblies) can be **unloaded**:

```csharp
// isCollectible: false — loaded forever (until process exits)
var permanent = new AssemblyLoadContext("Permanent", false);

// isCollectible: true — can be unloaded at runtime
var temporary = new AssemblyLoadContext("Plugins", true);
temporary.LoadFromAssemblyPath(pluginPath);
// ... use the plugin ...
temporary.Unload(); // assemblies become eligible for GC
```

| `isCollectible` | Behavior | Use case |
|---|---|---|
| `false` | Assembly stays loaded for the lifetime of the process | Core libraries, things you always need |
| `true` | Can call `Unload()` to release assemblies and types | Plugin systems, hot-reload scenarios |

```ad-note
`Unload()` doesn't free memory immediately — it marks the context for unloading. The assemblies and types are actually freed when the GC collects all references to them. You must release all references to types and instances from that context for it to fully unload.
```


---

## Default ALC vs Custom ALC

| Aspect | Default ALC | Custom ALC |
|---|---|---|
| Created by | Runtime automatically | You, via `new AssemblyLoadContext(...)` |
| Can be unloaded | No | Only if `isCollectible: true` |
| Type sharing | Types are directly usable everywhere | Types are isolated — casting across contexts fails |
| Static fields | One copy | Each context gets its own copy |
| Use case | Normal application code | Plugins, hot-reload, side-by-side versioning |
