---
tags:
 - csharp
 - appdomains
---

# AppDomain Overview

## What Is an AppDomain?

An Application Domain (AppDomain) is a **logical subdivision within a process** that acts as an isolation boundary for running .NET code. Think of it as a lightweight "process within a process."

In traditional unmanaged applications (C/C++), the executable is hosted directly in a Windows process. In .NET Framework, the executable is hosted inside an AppDomain, which itself lives inside the process:

```
┌─────────────────────────────────────────────────────┐
│                  Windows Process                    │
│                                                     │
│  ┌────────────────────┐  ┌────────────────────┐     │
│  │   AppDomain 1      │  │   AppDomain 2      │     │
│  │  (Default)         │  │  (Plugin)          │     │
│  │                    │  │                    │     │
│  │  ┌──────────────┐  │  │  ┌──────────────┐  │     │
│  │  │ Assembly A   │  │  │  │ Assembly C   │  │     │
│  │  │ Assembly B   │  │  │  │ Assembly D   │  │     │
│  │  └──────────────┘  │  │  └──────────────┘  │     │
│  └────────────────────┘  └────────────────────┘     │
│                                                     │
│  Shared: CLR runtime, OS resources                  │
└─────────────────────────────────────────────────────┘
```

Every .NET Framework process has at least one AppDomain — the **default AppDomain** — which is created automatically when the process starts.

```ad-danger
**AppDomains are a .NET Framework concept only.** They are NOT supported in .NET Core / .NET 5+. The modern replacement is `AssemblyLoadContext`. If you're working on .NET 5+, this page is historical context — see the section at the bottom.
```


---

## Why AppDomains Existed

AppDomains solved two real problems:

1. **How do you isolate code without the overhead of a full OS process?**
2. **How do you abstract away OS differences?** — AppDomains provided an OS-neutral hosting layer. Instead of the executable being loaded directly into a Windows process (which is OS-specific), it was loaded into an AppDomain — a logical construct managed by the CLR. This abstracted away the differences in how each underlying OS represents a loaded executable, making .NET more portable at the hosting level.

| Concern         | Separate Processes                       | AppDomains                                            |
| --------------- | ---------------------------------------- | ----------------------------------------------------- |
| Isolation       | Full — separate memory space             | Partial — separate logical space, same process memory |
| Overhead        | High — ~1MB+ per process, slow to create | Low — lightweight, fast to create/destroy             |
| Communication   | IPC needed (pipes, sockets)              | Objects can be passed (with marshaling)               |
| Fault tolerance | One crash doesn't affect others          | One crash can be contained within its AppDomain       |
| Unloading       | Kill the process                         | Unload just the AppDomain (assemblies + data freed)   |


---

## The Three Key Benefits

### 1. Isolation

AppDomains are **fully isolated** from each other. An application in one AppDomain cannot directly access data (global variables, static fields) in another AppDomain:

```csharp
// Static field in AppDomain 1
static int Counter = 42;

// AppDomain 2 has its OWN copy of Counter — they don't share.
// To communicate, you need marshaling or a distributed protocol.
```

### 2. Unloadability

You can unload an AppDomain at runtime, which releases all assemblies and memory associated with it — without restarting the entire process. This was essential for:
- **Plugin systems** — load a plugin, use it, unload it, load a newer version
- **Long-running servers** — IIS hosted each ASP.NET application in its own AppDomain and could recycle them independently

```csharp
// .NET Framework only
AppDomain pluginDomain = AppDomain.CreateDomain("Plugins");

// Load and use plugin assemblies...
pluginDomain.DoCallBack(() =>
{
    Console.WriteLine($"Running in: {AppDomain.CurrentDomain.FriendlyName}");
});

// Unload everything — assemblies, static data, all gone
AppDomain.Unload(pluginDomain);
```

### 3. Security Boundaries

Each AppDomain could run code with different **permission levels**. Untrusted code (downloaded plugins, user scripts) could be sandboxed in a restricted AppDomain while the host application ran with full trust.


---

## Cross-AppDomain Communication

Since AppDomains are isolated, passing data between them requires **marshaling** — objects must cross the boundary either by:

1. **Marshal by value** — the object is serialized, copied across, and deserialized (requires `[Serializable]`)
2. **Marshal by reference** — a proxy object is created in the calling domain that forwards calls across the boundary (requires inheriting `MarshalByRefObject`)

```csharp
// .NET Framework only
// A type that can be accessed across AppDomain boundaries
public class PluginWorker : MarshalByRefObject
{
    public string DoWork(string input)
    {
        return $"Processed '{input}' in {AppDomain.CurrentDomain.FriendlyName}";
    }
}

// Create a new AppDomain and get a proxy to a remote object
AppDomain domain = AppDomain.CreateDomain("WorkerDomain");
PluginWorker proxy = (PluginWorker)domain.CreateInstanceAndUnwrap(
    typeof(PluginWorker).Assembly.FullName,
    typeof(PluginWorker).FullName);

string result = proxy.DoWork("hello");  // call goes across the boundary
Console.WriteLine(result);

AppDomain.Unload(domain);
```

This marshaling overhead was one of the drawbacks — crossing AppDomain boundaries was significantly slower than a normal method call.


---

## Interacting with the Current AppDomain

Even if you never create additional AppDomains, the default AppDomain provides useful diagnostic info:

```csharp
AppDomain currentDomain = AppDomain.CurrentDomain;

Console.WriteLine($"Name:       {currentDomain.FriendlyName}");
Console.WriteLine($"Base Dir:   {currentDomain.BaseDirectory}");
Console.WriteLine($"ID:         {currentDomain.Id}");

// List all assemblies loaded in this AppDomain
foreach (var asm in currentDomain.GetAssemblies())
{
    Console.WriteLine($"  {asm.GetName().Name}");
}

// Subscribe to events
currentDomain.UnhandledException += (sender, e) =>
{
    Console.WriteLine($"Unhandled: {e.ExceptionObject}");
};
```

```ad-note
`AppDomain.CurrentDomain` still works in .NET 5+ — but it always returns the **single, default AppDomain**. You cannot create or unload additional ones.
```


---

## .NET Core / .NET 5+ — AppDomains Are Gone

In modern .NET, AppDomains were removed. There is always exactly **one AppDomain per process**, and you cannot create more. The reasons:

1. **Cross-platform complexity** — AppDomains relied heavily on Windows-specific OS features
2. **Performance cost** — cross-domain marshaling was slow
3. **`AssemblyLoadContext` replaced the key use case** — loading/unloading assemblies at runtime

### The Modern Replacement: AssemblyLoadContext

`AssemblyLoadContext` handles the main thing people actually used AppDomains for — dynamically loading and unloading assemblies (plugins):

```csharp
// .NET 5+ — load and unload assemblies without AppDomains
var context = new AssemblyLoadContext("Plugins", isCollectible: true);

// Load a plugin assembly
Assembly plugin = context.LoadFromAssemblyPath(@"C:\Plugins\MyPlugin.dll");
Type pluginType = plugin.GetType("MyPlugin.Worker");
object instance = Activator.CreateInstance(pluginType);

// Use the plugin...

// Unload all assemblies in this context
context.Unload();
// After GC collects, the assemblies and types are freed
```

### What's Gone vs. What's Still Available

| Feature | .NET Framework | .NET 5+ |
|---|---|---|
| Default AppDomain (`AppDomain.CurrentDomain`) | Yes | Yes (always exactly one) |
| Create additional AppDomains | Yes (`AppDomain.CreateDomain`) | No — throws `PlatformNotSupportedException` |
| Unload AppDomains | Yes (`AppDomain.Unload`) | No |
| Security sandboxing per AppDomain | Yes | No — use separate processes instead |
| Load/unload assemblies dynamically | Yes (via AppDomain) | Yes (via `AssemblyLoadContext`) |
| `UnhandledException` event | Yes | Yes |
| `GetAssemblies()` | Yes | Yes |

See also: [[Process]] for the OS-level isolation boundary that replaced AppDomain-level isolation in .NET 5+.
