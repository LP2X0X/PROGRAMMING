---
tags:
 - csharp
 - serialization
 - json
---

## System.Text.Json -- The Built-In JSON Library for Modern .NET

`System.Text.Json` is the JSON serialization library that ships with .NET Core 3.0+ and all subsequent .NET versions. It was built from scratch by Microsoft to replace the dependency on `Newtonsoft.Json` (Json.NET) with a high-performance, low-allocation, secure-by-default alternative that requires no NuGet package.

The library lives in two namespaces:

| Namespace | Contains |
|---|---|
| `System.Text.Json` | Core types: `JsonSerializer`, `JsonDocument`, `JsonElement`, `Utf8JsonReader`, `Utf8JsonWriter` |
| `System.Text.Json.Serialization` | Attributes (`[JsonPropertyName]`, `[JsonIgnore]`, etc.), `JsonConverter<T>`, source generation |

---

## Why System.Text.Json Replaced Newtonsoft.Json as the Default

Microsoft had specific engineering goals when building this library. Understanding them explains the design trade-offs you will encounter:

1. **Performance** -- `System.Text.Json` operates on raw UTF-8 bytes internally using `Span<T>` and `ReadOnlySpan<T>`. It avoids allocating intermediate `string` objects whenever possible. Benchmarks consistently show 1.5x to 3x faster than Newtonsoft.Json with significantly fewer allocations.

2. **Lower allocations** -- The `Utf8JsonReader` is a `ref struct` that reads directly from a `ReadOnlySpan<byte>`. No `string` is allocated until you explicitly call `.GetString()`. This matters enormously in hot paths like ASP.NET Core request/response handling.

3. **Built-in** -- No NuGet dependency. Ships with the runtime. This means the framework itself (ASP.NET Core, Minimal APIs, configuration) can use JSON serialization without taking an external dependency.

4. **Secure by default** -- Strict parsing rules, no support for dangerous features like type-name handling (which enabled BinaryFormatter-style deserialization attacks in Newtonsoft.Json's `TypeNameHandling.All`).

5. **AOT-friendly** -- Source generators (.NET 6+) enable trimming and Native AOT compilation by eliminating reflection.

> [!ad-note] When to Still Use Newtonsoft.Json
> Despite `System.Text.Json` being the default, there are legitimate reasons to use Newtonsoft.Json:
> - Your project already uses it extensively and migration would be costly
> - You need features STJ still lacks (e.g., `JsonPath` queries, `JToken`-style dynamic manipulation with full feature parity)
> - You need lenient parsing of messy/non-standard JSON from third parties
> - You are targeting .NET Framework (where STJ has limited support)
>
> In new projects, prefer `System.Text.Json` unless you hit a specific limitation.

---

## Key Types at a Glance

| Type | Purpose | When to Use |
|---|---|---|
| `JsonSerializer` | Static class for serializing/deserializing POCOs | 90% of use cases -- you have a C# class that maps to your JSON |
| `JsonSerializerOptions` | Configuration object for `JsonSerializer` | Customize naming, null handling, converters, etc. |
| `JsonDocument` | Read-only DOM -- parses JSON into a tree you can query | Parse JSON when you don't have (or don't want) a matching C# class |
| `JsonElement` | A single value inside a `JsonDocument` | Navigate a `JsonDocument` tree: `.GetProperty()`, `.GetString()`, etc. |
| `JsonNode` | Mutable DOM (.NET 6+) -- parse, modify, and re-serialize | When you need to read JSON, change some values, and write it back |
| `JsonObject` | A mutable JSON object (child of `JsonNode`) | Work with `{ }` objects in the mutable DOM |
| `JsonArray` | A mutable JSON array (child of `JsonNode`) | Work with `[ ]` arrays in the mutable DOM |
| `JsonValue` | A mutable JSON primitive (child of `JsonNode`) | Work with individual values in the mutable DOM |
| `Utf8JsonReader` | Low-level, forward-only, zero-alloc UTF-8 reader (`ref struct`) | Maximum performance, streaming, or building a custom parser |
| `Utf8JsonWriter` | Low-level JSON writer over `IBufferWriter<byte>` or `Stream` | Maximum performance, streaming, or custom output format |
| `JsonConverter<T>` | Base class for custom serialization logic | When the default serialization of a type is wrong or insufficient |

---

## Three Levels of API

`System.Text.Json` offers three distinct abstraction levels. Choosing the right one depends on your scenario:

### Level 1: High-Level -- `JsonSerializer` (Most Common)

You have a C# class. You want JSON. Or you have JSON and you want a C# object. This is what you use 90% of the time.

```csharp
// Serialize
var person = new Person { Name = "Long", Age = 28 };
string json = JsonSerializer.Serialize(person);

// Deserialize
Person? p = JsonSerializer.Deserialize<Person>(json);
```

See [[JsonSerializer]] for the full API reference.

### Level 2: Mid-Level -- `JsonDocument` (Read-Only) / `JsonNode` (Mutable)

You have JSON but no C# class to map it to, or you only need a few values from a large JSON payload.

**JsonDocument (read-only, .NET Core 3.0+):**

```csharp
// Parse JSON and extract specific values without a class
using JsonDocument doc = JsonDocument.Parse(json);
JsonElement root = doc.RootElement;

string name = root.GetProperty("name").GetString()!;    // "Long"
int age = root.GetProperty("age").GetInt32();            // 28

// Enumerate an array
foreach (JsonElement item in root.GetProperty("hobbies").EnumerateArray())
{
    Console.WriteLine(item.GetString());
}
```

> [!ad-important] JsonDocument Is IDisposable
> `JsonDocument` rents memory from `ArrayPool<byte>` internally. You **must** dispose it (use a `using` statement) or you will leak pooled arrays. This is a common mistake.

> [!ad-warning] JsonElement Does Not Survive JsonDocument Disposal
> A `JsonElement` is a view into the `JsonDocument`'s memory. Once the `JsonDocument` is disposed, any `JsonElement` obtained from it becomes invalid. If you need to keep a value beyond the `using` block, call `element.Clone()` to create an independent copy.

**JsonNode (mutable, .NET 6+):**

```csharp
// Parse, modify, re-serialize
JsonNode? node = JsonNode.Parse(json);
node!["name"] = "Updated Name";                    // modify a property
node!["newProp"] = 42;                             // add a property
((JsonObject)node).Remove("age");                  // remove a property

string updatedJson = node.ToJsonString();          // re-serialize
```

`JsonNode` is the mutable counterpart to `JsonDocument`. Use it when you need to parse JSON, change some values, and write it back -- without defining a C# class.

### Level 3: Low-Level -- `Utf8JsonReader` / `Utf8JsonWriter`

For maximum performance and minimum allocations. You manually walk through JSON tokens. This is the API that `JsonSerializer` and `JsonDocument` are built on top of.

```csharp
// ---- Low-level reading ----
byte[] utf8Json = Encoding.UTF8.GetBytes(json);
var reader = new Utf8JsonReader(utf8Json);

while (reader.Read())
{
    switch (reader.TokenType)
    {
        case JsonTokenType.PropertyName:
            Console.Write($"{reader.GetString()}: ");
            break;
        case JsonTokenType.String:
            Console.WriteLine(reader.GetString());
            break;
        case JsonTokenType.Number:
            Console.WriteLine(reader.GetInt32());
            break;
    }
}

// ---- Low-level writing ----
using var stream = new MemoryStream();
using var writer = new Utf8JsonWriter(stream, new JsonWriterOptions { Indented = true });

writer.WriteStartObject();
writer.WriteString("name", "Long");
writer.WriteNumber("age", 28);
writer.WriteStartArray("hobbies");
writer.WriteStringValue("coding");
writer.WriteStringValue("reading");
writer.WriteEndArray();
writer.WriteEndObject();

writer.Flush();
string result = Encoding.UTF8.GetString(stream.ToArray());
```

> [!ad-note] When to Drop to Low-Level
> You should only use `Utf8JsonReader`/`Utf8JsonWriter` when:
> - You need to process very large JSON files in a streaming fashion (too large to fit in memory)
> - You are writing a custom `JsonConverter<T>` (converters receive a `Utf8JsonReader` in their `Read` method)
> - You are in a hot path where even `JsonSerializer`'s allocations are too much
> - You need to write JSON without creating an intermediate object
>
> For all other cases, `JsonSerializer` or `JsonNode` will be simpler and sufficient.

---

## Decision Guide: Which API Level?

| Scenario | Use | Why |
|---|---|---|
| Serialize/deserialize a known C# class | `JsonSerializer` | Simplest, handles mapping automatically |
| Parse JSON without a class, read-only access | `JsonDocument` + `JsonElement` | Lightweight, no class definition needed |
| Parse and **modify** JSON without a class | `JsonNode` (.NET 6+) | Mutable DOM, natural syntax |
| Process huge JSON streams | `Utf8JsonReader` / `Utf8JsonWriter` | Forward-only, zero-copy, constant memory |
| Build a custom converter | `Utf8JsonReader` + `Utf8JsonWriter` | Required by the `JsonConverter<T>` contract |
| Need `JsonPath` queries | Consider `Newtonsoft.Json` | STJ has no built-in `JsonPath` support |

---

## System.Text.Json vs Newtonsoft.Json -- Detailed Comparison

| Feature | System.Text.Json | Newtonsoft.Json |
|---|---|---|
| **Performance** | 1.5-3x faster, significantly fewer allocations | Slower, more GC pressure |
| **Built-in** | Yes (.NET Core 3.0+) | No -- NuGet package `Newtonsoft.Json` |
| **Strictness** | Strict by default (rejects comments, trailing commas, unquoted names) | Lenient by default (accepts many non-standard forms) |
| **Default naming** | PascalCase (matches C# conventions) | camelCase |
| **Case-insensitive matching** | Opt-in via `PropertyNameCaseInsensitive` | On by default |
| **Polymorphism** | `[JsonDerivedType]` (.NET 7+) | `TypeNameHandling` (security risk if misused) |
| **Custom converters** | `JsonConverter<T>` -- receives `Utf8JsonReader`/`Utf8JsonWriter` | `JsonConverter` -- receives `JsonReader`/`JsonWriter` |
| **Dynamic/untyped access** | `JsonDocument` (read-only) / `JsonNode` (mutable, .NET 6+) | `JObject`, `JArray`, `JToken` -- very mature |
| **JsonPath queries** | Not supported natively | Fully supported via `SelectToken`, `SelectTokens` |
| **Source generation / AOT** | Yes (.NET 6+) | No |
| **`required` keyword support** | Yes (.NET 7+) | Limited |
| **Comments in JSON** | Opt-in via `ReadCommentHandling.Skip` | Supported by default |
| **Circular references** | `ReferenceHandler.IgnoreCycles` (.NET 6+) or `Preserve` | `ReferenceLoopHandling.Ignore` / `Serialize` |
| **Global settings** | No global -- must pass `JsonSerializerOptions` | `JsonConvert.DefaultSettings` |
| **LINQ to JSON** | No equivalent | `JObject.Parse(json).SelectToken("$.path")` |

> [!ad-warning] Migration Pitfalls
> If you are migrating from Newtonsoft.Json to System.Text.Json, be aware of behavioral differences that can cause silent bugs:
> - **Naming**: Newtonsoft defaults to camelCase; STJ defaults to PascalCase. Your API consumers may break if you switch without setting `PropertyNamingPolicy = JsonNamingPolicy.CamelCase`.
> - **Case sensitivity**: Newtonsoft is case-insensitive by default; STJ is case-sensitive. Set `PropertyNameCaseInsensitive = true` for compatibility.
> - **Missing properties**: Newtonsoft silently ignores missing properties; STJ also ignores them by default, but `[JsonRequired]` (.NET 7+) or `UnmappedMemberHandling.Disallow` (.NET 8+) can change this.
> - **Number handling**: Newtonsoft reads `"123"` (string) as a number; STJ throws by default. Use `JsonNumberHandling.AllowReadingFromString` for compatibility.
> - **Enums**: Newtonsoft serializes enums as integers by default but has `StringEnumConverter`; STJ serializes as integers by default and requires `JsonStringEnumConverter` to get strings.

---

## Source Generators (.NET 6+) -- AOT and Trimming Support

By default, `System.Text.Json` uses **reflection** at runtime to discover properties, constructors, and attributes on your types. This works fine but has two drawbacks:

1. **Startup cost** -- reflection metadata must be built the first time a type is serialized
2. **Incompatible with AOT** -- Native AOT compilation and aggressive trimming break reflection

Source generators solve this by generating serialization metadata **at compile time**:

```csharp
// Step 1: Define a source generation context
[JsonSourceGenerationOptions(WriteIndented = true)]
[JsonSerializable(typeof(Person))]
[JsonSerializable(typeof(List<Person>))]
internal partial class AppJsonContext : JsonSerializerContext
{
}

// Step 2: Use the generated context
string json = JsonSerializer.Serialize(person, AppJsonContext.Default.Person);
Person? p = JsonSerializer.Deserialize(json, AppJsonContext.Default.Person);
```

> [!ad-info] When Source Generators Matter
> - **You are publishing a Native AOT app** -- source generators are required; reflection-based serialization will not work.
> - **You are trimming your app** -- source generators prevent the trimmer from removing types that serialization needs.
> - **You want faster startup** -- source generators eliminate reflection overhead on first use.
> - **You are writing a library** -- source generators make your library compatible with AOT consumers.
>
> For typical server applications (.NET 8+ with default settings), the reflection-based API works fine and source generators are optional.

---

## Supported Types Out of the Box

`System.Text.Json` can serialize and deserialize these types without any custom converters:

| Category | Types |
|---|---|
| **Primitives** | `bool`, `byte`, `sbyte`, `short`, `ushort`, `int`, `uint`, `long`, `ulong`, `float`, `double`, `decimal` |
| **Strings** | `string`, `char` |
| **Date/Time** | `DateTime`, `DateTimeOffset`, `DateOnly` (.NET 6+), `TimeOnly` (.NET 6+), `TimeSpan` (.NET 8+) |
| **GUIDs & URIs** | `Guid`, `Uri` |
| **Enums** | All enums (as integers by default; use `JsonStringEnumConverter` for string names) |
| **Nullable** | `Nullable<T>` for all above |
| **Collections** | `T[]`, `List<T>`, `Dictionary<string, T>`, `HashSet<T>`, `Queue<T>`, `Stack<T>`, `IEnumerable<T>`, `IReadOnlyList<T>`, and most `System.Collections.Generic` types |
| **Objects** | POCOs with public properties and a parameterless constructor (or `[JsonConstructor]`) |
| **Special** | `JsonElement`, `JsonDocument`, `JsonNode`, `object` (limited -- serializes as-is, deserializes as `JsonElement`) |

> [!ad-warning] `object` Deserialization Trap
> If a property is typed as `object`, `System.Text.Json` deserializes JSON values into `JsonElement`, not the "obvious" .NET types. A JSON number `42` becomes a `JsonElement` of kind `Number`, not a boxed `int`. This catches many people off guard coming from Newtonsoft.Json, which would produce a boxed `long`.

---

## Related Notes

- [[Object Serialization Overview]] -- What serialization is, why it matters, format comparison
- [[JsonSerializer]] -- Full API reference for the high-level serialization methods
- [[JsonSerializerOptions]] -- Configuring naming, nulls, circular references, converters, and more
- [[Serialization Attributes]] -- Per-property and per-type attribute reference
