---
tags:
  - csharp
  - serialization
  - security
---

## BinaryFormatter and Modern Alternatives

**`BinaryFormatter`** was a .NET serializer that converted object graphs to a compact binary format. It was powerful -- it could serialize private fields, circular references, and complex inheritance hierarchies with minimal developer effort. It was also one of the most dangerous APIs in the entire .NET ecosystem. This note covers what it was, why it was removed, and what to use instead.

---

## What BinaryFormatter Was

`BinaryFormatter` (in `System.Runtime.Serialization.Formatters.Binary`) could serialize nearly any .NET object to a binary stream and reconstruct it later:

```csharp
// OLD CODE -- DO NOT USE -- shown for historical context only
// var formatter = new BinaryFormatter();
//
// // Serialize
// using var stream = new MemoryStream();
// formatter.Serialize(stream, myObject);
//
// // Deserialize
// stream.Position = 0;
// var restored = (MyClass)formatter.Deserialize(stream);
```

It required the `[Serializable]` attribute on the class:

```csharp
// [Serializable]
// public class Person
// {
//     public string Name;
//     private int _age;        // Private fields WERE serialized
//     public List<Person> Friends;  // Circular references handled
// }
```

### Capabilities That Made It Popular

- Serialized **private fields** automatically (no attributes needed per member)
- Handled **circular references** in object graphs
- Supported **inheritance** transparently
- Required minimal code -- just `[Serializable]` and two method calls
- Used for .NET Remoting, `ViewState` in ASP.NET, clipboard operations, deep cloning

---

## Why It Was Removed

### The Obsolescence Timeline

| .NET Version | Status |
|---|---|
| .NET Framework (all) | Fully available, widely used |
| .NET 5 | Marked `[Obsolete]` with a warning |
| .NET 7 | Disabled by default in most project types |
| .NET 8 | `Deserialize` **throws `NotSupportedException`** by default |
| .NET 9 | **Completely removed** from the runtime |

> [!ad-warning] BinaryFormatter is a critical security vulnerability
> `BinaryFormatter` is not just deprecated for being old -- it is **fundamentally unsafe** and cannot be fixed. Microsoft's official stance is that **no amount of configuration or filtering makes it safe** for untrusted input. Every use of `BinaryFormatter.Deserialize` on data you don't fully control is a potential Remote Code Execution (RCE) vulnerability.

### The Security Problem Explained

The core issue is that `BinaryFormatter` embeds **full type information** in the serialized payload. When deserializing, it instantiates whatever types the payload requests and sets their fields to whatever values the payload specifies.

Here's why that's catastrophic:

1. **The payload controls which types are instantiated** -- an attacker can specify any type available in the process
2. **The payload controls field values** -- the attacker sets the state of those objects
3. **Some .NET types execute code in constructors, property setters, finalizers, or `Dispose` methods** -- the attacker chains these together
4. **The result: arbitrary code execution** -- by carefully choosing types and values, the attacker can execute any command on the server

```
Attacker crafts binary payload
    --> BinaryFormatter.Deserialize() processes it
    --> Instantiates types chosen by the attacker
    --> Sets fields to values chosen by the attacker
    --> .NET type's constructor/finalizer/Dispose runs attacker's logic
    --> Arbitrary code executes on the server (RCE)
```

> [!ad-important] This is not theoretical
> Real-world exploits using `BinaryFormatter` deserialization have been found in production systems. Libraries of "gadget chains" (sequences of types that chain together to execute arbitrary code) are publicly available. Tools like `ysoserial.net` can generate exploit payloads automatically.

### Why It Can't Be Fixed

Some developers ask: "Can't you just filter which types are allowed?" Microsoft's answer is no:

- The set of dangerous types changes as the framework evolves
- New gadget chains are discovered regularly
- Even "safe" types can become dangerous when combined
- A deny-list approach will always miss new attack vectors
- An allow-list approach is fragile and still vulnerable to type confusion

---

## What to Use Instead

### Decision Guide

| Scenario | Recommended Replacement |
|---|---|
| General-purpose serialization | **`System.Text.Json`** (built-in, fast, secure) |
| Complex JSON needs (polymorphism, circular refs) | **`Newtonsoft.Json`** (NuGet, mature, flexible) |
| XML serialization / legacy APIs | **`XmlSerializer`** or **`DataContractSerializer`** |
| High-performance binary (cross-platform) | **MessagePack** or **Protobuf-net** (NuGet) |
| High-performance binary (.NET only) | **MemoryPack** (NuGet, .NET 7+) |
| Inter-process communication (IPC) | **gRPC with Protocol Buffers** |
| Deep cloning objects | Manual copy constructor or serialize/deserialize via `System.Text.Json` |
| Clipboard / drag-and-drop in WinForms/WPF | `System.Text.Json` to string, then use string-based clipboard APIs |

---

## Modern Binary Serialization Libraries

When JSON or XML is too slow or too large, these binary serializers are the modern replacements.

### MessagePack (MessagePack-CSharp)

**MessagePack** is a compact binary format that is cross-platform (works with Python, JavaScript, Go, etc.) and significantly faster than JSON.

```csharp
// NuGet: MessagePack
using MessagePack;

[MessagePackObject]
public class Person
{
    [Key(0)]  // Index-based keys for compact output
    public string Name { get; set; } = "";

    [Key(1)]
    public int Age { get; set; }

    [IgnoreMember]  // Exclude from serialization
    public string InternalCode { get; set; } = "";
}

// Serialize to byte[]
byte[] bytes = MessagePackSerializer.Serialize(person);

// Deserialize from byte[]
Person restored = MessagePackSerializer.Deserialize<Person>(bytes);

// Serialize to JSON (for debugging)
string json = MessagePackSerializer.SerializeToJson(person);
```

| Feature | Detail |
|---|---|
| **NuGet package** | `MessagePack` |
| **Speed** | Very fast (near-zero allocation with `Memory<byte>`) |
| **Size** | ~50-80% smaller than JSON |
| **Cross-platform** | Yes -- MessagePack is a standard format |
| **Schema** | Attribute-based (`[Key]`) or contractless mode |
| **.NET versions** | .NET Standard 2.0+ |

### Protobuf-net

**Protobuf-net** is a .NET implementation of Google's Protocol Buffers. It can work schema-first (`.proto` files) or attribute-first (C# attributes).

```csharp
// NuGet: protobuf-net
using ProtoBuf;

[ProtoContract]
public class Person
{
    [ProtoMember(1)]
    public string Name { get; set; } = "";

    [ProtoMember(2)]
    public int Age { get; set; }
}

// Serialize
using var stream = new MemoryStream();
Serializer.Serialize(stream, person);
byte[] bytes = stream.ToArray();

// Deserialize
stream.Position = 0;
Person restored = Serializer.Deserialize<Person>(stream);
```

| Feature | Detail |
|---|---|
| **NuGet package** | `protobuf-net` |
| **Speed** | Very fast |
| **Size** | Very compact (varint encoding, field tags) |
| **Cross-platform** | Yes -- Protocol Buffers is a Google standard |
| **Schema** | Attribute-based or `.proto` file |
| **Use with gRPC** | Native fit -- gRPC uses protobuf as its wire format |

### MemoryPack (.NET 7+)

**MemoryPack** is the fastest .NET binary serializer, created by Yoshifumi Kawai (the creator of MessagePack-CSharp). It achieves zero-encoding overhead by leveraging C# source generators.

```csharp
// NuGet: MemoryPack
using MemoryPack;

[MemoryPackable]
public partial class Person  // Must be partial for source generator
{
    public string Name { get; set; } = "";
    public int Age { get; set; }

    [MemoryPackIgnore]
    public string InternalCode { get; set; } = "";
}

// Serialize to byte[]
byte[] bytes = MemoryPackSerializer.Serialize(person);

// Deserialize from byte[]
Person? restored = MemoryPackSerializer.Deserialize<Person>(bytes);
```

| Feature | Detail |
|---|---|
| **NuGet package** | `MemoryPack` |
| **Speed** | Fastest .NET serializer (zero-encoding, source-generated) |
| **Size** | Very compact |
| **Cross-platform** | .NET only (not interoperable with other languages) |
| **Schema** | Attribute-based with source generator |
| **.NET versions** | .NET 7+ only |

> [!ad-note] Cross-platform vs .NET-only
> If your binary data only stays within .NET services, **MemoryPack** gives the best performance. If you need to exchange data with non-.NET systems (Python, Go, JavaScript), use **MessagePack** or **Protobuf-net**.

### Comparison of Binary Serializers

| | MessagePack | Protobuf-net | MemoryPack |
|---|---|---|---|
| **Speed** | Very fast | Very fast | Fastest |
| **Payload size** | Small | Smallest | Small |
| **Cross-platform** | Yes | Yes | No (.NET only) |
| **Source generator** | Optional | Optional | Required |
| **Min .NET version** | .NET Standard 2.0 | .NET Standard 2.0 | .NET 7 |
| **gRPC integration** | No (separate format) | Native (protobuf is gRPC's format) | No |
| **Community** | Large | Large | Growing |

---

## Migration: BinaryFormatter to System.Text.Json

The most common migration path is from `BinaryFormatter` to `System.Text.Json`, since it requires no NuGet packages:

### Before (BinaryFormatter)

```csharp
// [Serializable]
// public class AppSettings
// {
//     public string Theme = "dark";
//     private int _fontSize = 14;    // Private field -- was serialized
//     public List<string> RecentFiles = new();
// }
//
// // Save
// var formatter = new BinaryFormatter();
// using var writeStream = File.Create("settings.dat");
// formatter.Serialize(writeStream, settings);
//
// // Load
// using var readStream = File.OpenRead("settings.dat");
// var settings = (AppSettings)formatter.Deserialize(readStream);
```

### After (System.Text.Json)

```csharp
using System.Text.Json;

public class AppSettings
{
    public string Theme { get; set; } = "dark";
    public int FontSize { get; set; } = 14;        // Must be public property now
    public List<string> RecentFiles { get; set; } = new();
}

// Configure options (optional)
var options = new JsonSerializerOptions
{
    WriteIndented = true  // Human-readable file
};

// Save
await using var writeStream = File.Create("settings.json");
await JsonSerializer.SerializeAsync(writeStream, settings, options);

// Load
await using var readStream = File.OpenRead("settings.json");
AppSettings? settings = await JsonSerializer.DeserializeAsync<AppSettings>(readStream);
```

### Key Differences When Migrating

| BinaryFormatter behavior | System.Text.Json equivalent |
|---|---|
| `[Serializable]` attribute | Not needed (no attribute required by default) |
| Private fields serialized | Must convert to public properties (or use `[JsonInclude]` on public fields) |
| Circular references handled | Set `options.ReferenceHandler = ReferenceHandler.Preserve` |
| `ISerializable` interface | Implement a custom `JsonConverter<T>` |
| `[NonSerialized]` on a field | `[JsonIgnore]` on a property |
| `[OnDeserializing]` callbacks | No direct equivalent -- use a custom converter or post-deserialization init |
| Binary output format | JSON text (or use MessagePack/MemoryPack for binary) |

> [!ad-warning] Private fields require refactoring
> The biggest pain point in migration is that `BinaryFormatter` serialized private fields automatically, but `System.Text.Json` does not. You must either:
> 1. Convert private fields to public properties (preferred)
> 2. Use `[JsonInclude]` on public fields
> 3. Write a custom `JsonConverter<T>` that accesses private state
>
> If your serialized data is persisted (files, databases), you'll also need a one-time migration to convert the binary format to JSON.

---

## Deep Cloning Without BinaryFormatter

A common use of `BinaryFormatter` was deep cloning objects. Here are safe alternatives:

### Using System.Text.Json

```csharp
public static T DeepClone<T>(T source)
{
    // Serialize to JSON bytes and immediately deserialize
    // This creates an independent copy of the entire object graph
    byte[] bytes = JsonSerializer.SerializeToUtf8Bytes(source);
    return JsonSerializer.Deserialize<T>(bytes)!;
}

// Usage
var original = new Person { Name = "Long", Age = 28 };
var clone = DeepClone(original);
clone.Name = "Modified";  // Does not affect original
```

### Manual Copy Constructor (Best Performance)

```csharp
public class Person
{
    public string Name { get; set; } = "";
    public int Age { get; set; }
    public Address Address { get; set; } = new();

    // Deep clone via copy constructor
    public Person DeepCopy()
    {
        return new Person
        {
            Name = this.Name,          // string is immutable -- safe to share
            Age = this.Age,            // value type -- copied automatically
            Address = new Address      // reference type -- must create new instance
            {
                City = this.Address.City,
                Zip = this.Address.Zip
            }
        };
    }
}
```

> [!ad-note] Performance comparison for deep cloning
> Manual copy is the fastest (no serialization overhead). `System.Text.Json` round-trip is convenient but slower. For performance-critical paths with complex objects, consider MemoryPack or a dedicated cloning library.

---

## See Also

- [[Custom JsonConverters]] -- writing custom converters for `System.Text.Json` (useful during migration)
- [[JsonDocument and JsonNode]] -- working with JSON when you don't have a class definition
- [[XML Serialization]] -- XML-based serialization with `XmlSerializer` and `DataContractSerializer`
