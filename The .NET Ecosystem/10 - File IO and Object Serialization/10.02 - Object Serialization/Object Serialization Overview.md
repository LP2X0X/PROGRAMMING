---
tags:
 - csharp
 - serialization
 - json
---

## Object Serialization -- Converting Object Graphs to Storable/Transmittable Formats

**Serialization** is the process of converting an in-memory object (and all the objects it references) into a format that can be stored on disk, sent over a network, or otherwise persisted outside the running process. **Deserialization** is the reverse -- reconstructing the live object from that stored format.

The key phrase is "[[Object Graphs|object graph]]." When you serialize an object, you are not just serializing that single object in isolation. Every object it references through its properties and fields is *also* serialized, and every object *those* reference, and so on recursively, until the entire reachable graph has been captured. This is what makes serialization powerful -- and what makes it dangerous if the graph is unexpectedly large or contains circular references.

```
          Serialization
 Object ─────────────────────> Bytes / Text (JSON, XML, binary)
 Graph   <─────────────────────
          Deserialization
```

---

## Why Serialize?

Serialization is not a single-purpose tool. It appears in virtually every category of .NET application:

| Purpose                          | Example                                                                      |
| -------------------------------- | ---------------------------------------------------------------------------- |
| **Persistence**                  | Save application state to a file so it survives restarts                     |
| **Network transmission**         | Send a C# object as a JSON payload to a REST API                             | 
| **Caching**                      | Store a computed result in Redis / disk cache to avoid recomputation         |
| **Deep cloning**                 | Serialize then deserialize to produce an independent copy of a complex graph |
| **Cross-platform data exchange** | Produce JSON/XML that a Python, Java, or JavaScript client can consume       |
| **Message queues**               | Publish an event object to RabbitMQ, Kafka, or Azure Service Bus             |
| **Configuration**                | Read/write `appsettings.json` or XML config files                            |
| **Logging & diagnostics**        | Serialize an object to include its full state in a log message               |

> [!ad-note] Deep Cloning via Serialization
> Serializing an object to JSON and immediately deserializing it back produces a completely independent copy -- no shared references with the original. This is a common trick for deep cloning complex object graphs when implementing `ICloneable` manually would be tedious and error-prone. The trade-off is performance: it is significantly slower than a hand-written deep copy.

---

## The Object Graph Concept

When you call `JsonSerializer.Serialize(order)` on an `Order` object, the serializer does not just look at `Order`'s own properties. It walks the entire reachable graph:

```csharp
public class Order
{
    public int Id { get; set; }
    public Customer Customer { get; set; }          // reference to another object
    public List<OrderLine> Lines { get; set; }       // reference to a collection of objects
}

public class OrderLine
{
    public Product Product { get; set; }             // another reference
    public int Quantity { get; set; }
}
```

Serializing an `Order` means serializing the `Customer`, every `OrderLine`, every `Product` inside every line, and any objects *those* reference. The serializer traverses the graph depth-first and writes each object it encounters.

> [!ad-warning] Circular References Will Throw by Default
> If `Customer` has a `List<Order>` property that refers back to the same `Order`, you have a **circular reference**. `System.Text.Json` will throw a `JsonException` by default. You must either break the cycle with `[JsonIgnore]` or enable `ReferenceHandler.IgnoreCycles` in [[JsonSerializerOptions]]. Newtonsoft.Json has similar options via `ReferenceLoopHandling`.

> [!ad-warning] Unexpected Graph Size
> If your object graph is deeper or wider than you expect (e.g., an ORM entity with lazy-loaded navigation properties that pull in the entire database), serialization can produce enormous output or throw a `StackOverflowException`. Always be deliberate about what gets serialized -- use DTOs (Data Transfer Objects) rather than serializing ORM entities directly.

---

## Serialization Formats in .NET

.NET supports multiple serialization formats, each with different trade-offs:

| Format | Library | Human-Readable? | Performance | Typical Use Case |
|---|---|---|---|---|
| **JSON** | `System.Text.Json` | Yes | Fast | Web APIs, config files, data exchange |
| **JSON** | `Newtonsoft.Json` (Json.NET) | Yes | Moderate | Legacy APIs, advanced scenarios STJ doesn't cover |
| **XML** | `XmlSerializer` | Yes | Moderate | SOAP services, legacy APIs, config files |
| **XML** | `DataContractSerializer` | Yes | Moderate | WCF services, SOAP, complex type hierarchies |
| **Binary (legacy)** | `BinaryFormatter` | No | Fast | ==OBSOLETE -- security risk, removed in .NET 9== |
| **Binary (modern)** | MessagePack-CSharp | No | Very fast | High-performance internal communication |
| **Binary (modern)** | Protobuf-net | No | Very fast | gRPC, cross-language services |
| **Binary (modern)** | MemoryPack | No | Extremely fast | .NET-to-.NET zero-copy scenarios |

> [!ad-info] Choosing a Format
> - **JSON** is the default choice for 90%+ of scenarios: human-readable, universally understood, good tooling, acceptable performance.
> - **XML** is for legacy integration, SOAP, or when your consumers require it.
> - **Modern binary** formats are for high-throughput internal services where human readability does not matter and you need maximum speed/minimum size.
> - **BinaryFormatter** is for nothing. Do not use it.

---

## The Evolution of .NET Serialization

Understanding the history helps you recognize legacy patterns in older codebases and understand why the ecosystem looks the way it does today.

### .NET Framework Era (2002-2016)

- **BinaryFormatter** -- Serialized the complete object graph including private fields. Produced compact binary output. Tightly coupled to .NET types -- could not be consumed by non-.NET systems. Had severe security vulnerabilities (deserialization attacks).
- **SoapFormatter** -- Like BinaryFormatter but in SOAP XML format. Also obsolete.
- **XmlSerializer** -- Serialized public properties to XML. Still alive and widely used. Cannot handle interfaces, abstract types, or private members without extra work.
- **DataContractSerializer** -- Introduced with WCF (.NET 3.0). Used `[DataContract]` and `[DataMember]` attributes. More flexible than `XmlSerializer` for complex type hierarchies.
- **Newtonsoft.Json (Json.NET)** -- James Newton-King's third-party library. Became the de facto standard for JSON in .NET. Bundled with ASP.NET Web API and later ASP.NET MVC. Extremely feature-rich and battle-tested.

### .NET Core / Modern .NET Era (2016-present)

- **System.Text.Json** -- Introduced in .NET Core 3.0 (2019) as the built-in JSON library. Designed for performance and security. Replaced Newtonsoft.Json as the default in ASP.NET Core. See [[System.Text.Json Overview]] for full details.
- **BinaryFormatter deprecated** -- Marked `[Obsolete]` in .NET 7, disabled by default in .NET 8, **removed entirely in .NET 9**.
- **Source generators** -- .NET 6+ added compile-time source generation for `System.Text.Json`, enabling AOT (Ahead-of-Time) compilation and eliminating reflection at runtime.

```
Timeline:
2002  BinaryFormatter, XmlSerializer              (.NET Framework 1.0)
2006  DataContractSerializer                       (.NET 3.0 / WCF)
2008  Newtonsoft.Json 1.0 released                 (third-party)
2013  Json.NET bundled with ASP.NET Web API        (de facto standard)
2019  System.Text.Json ships in .NET Core 3.0      (built-in replacement)
2022  STJ source generators                        (.NET 6)
2023  BinaryFormatter marked [Obsolete]            (.NET 7)
2024  BinaryFormatter removed                      (.NET 9)
```

---

## BinaryFormatter -- Why It Is Dangerous

> [!ad-warning] BinaryFormatter Is OBSOLETE and a Security Risk
> `BinaryFormatter` was removed in .NET 9 and should **never** be used for deserializing untrusted data, even in older .NET versions. The fundamental problem is that `BinaryFormatter` embeds full type information in the serialized payload. An attacker can craft a malicious payload that causes the deserializer to instantiate arbitrary types, execute code during construction, and achieve **Remote Code Execution (RCE)**.
>
> This is not a theoretical risk. Real-world exploits have been demonstrated against applications that deserialize untrusted `BinaryFormatter` data. Microsoft's official guidance is: **do not use BinaryFormatter under any circumstances**.
>
> **Replacements:**
> - For JSON: `System.Text.Json` (see [[System.Text.Json Overview]])
> - For binary: MessagePack-CSharp, Protobuf-net, or MemoryPack
> - For cross-process .NET: `System.Text.Json` with known types

If you encounter `BinaryFormatter` in a legacy codebase, migrating away from it should be a high priority. Microsoft provides a migration guide at `learn.microsoft.com/en-us/dotnet/standard/serialization/binaryformatter-migration-guide`.

---

## Quick Example -- The Core Concept

The simplest possible serialization round-trip with `System.Text.Json`:

```csharp
using System.Text.Json;

// Define a simple POCO (Plain Old CLR Object)
var person = new Person { Name = "Long", Age = 28 };

// ---- Serialize: Object --> JSON string ----
string json = JsonSerializer.Serialize(person);
Console.WriteLine(json);
// Output: {"Name":"Long","Age":28}

// ---- Deserialize: JSON string --> Object ----
Person? restored = JsonSerializer.Deserialize<Person>(json);
Console.WriteLine($"{restored?.Name}, {restored?.Age}");
// Output: Long, 28

public class Person
{
    public string Name { get; set; } = string.Empty;
    public int Age { get; set; }
}
```

> [!ad-note] The `?` on the Deserialized Type
> `JsonSerializer.Deserialize<T>()` returns `T?` (nullable) because the JSON string `"null"` is valid JSON and would produce a `null` result. Always null-check or use the null-forgiving operator only when you are certain the input is not null.

For a deep dive into the serializer API, see [[JsonSerializer]]. For configuration options, see [[JsonSerializerOptions]]. For per-property control with attributes, see [[Serialization Attributes]].

---

## Notes in This Folder

- [[System.Text.Json Overview]] -- Architecture and design of the built-in JSON library
- [[JsonSerializer]] -- The high-level serialize/deserialize API in detail
- [[JsonSerializerOptions]] -- Configuring serialization behavior (naming, nulls, references, etc.)
- [[Serialization Attributes]] -- Per-property and per-type attribute reference
