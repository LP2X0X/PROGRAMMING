---
tags:
  - csharp
  - serialization
  - json
---

## JsonDocument and JsonNode

Sometimes you need to work with JSON without mapping it to a C# class. Maybe the JSON structure is unknown at compile time, varies between responses, or you only need to read a few fields from a large payload. **`System.Text.Json`** provides two DOM (Document Object Model) APIs for this:

- **`JsonDocument`** -- read-only, high-performance, pooled memory
- **`JsonNode`** -- mutable, flexible, GC-managed

> [!ad-info] When to use DOM APIs vs. deserialization
> Use `JsonSerializer.Deserialize<T>()` when you **have** a class that matches the JSON shape and you need all (or most) of its data. Use the DOM APIs when you **don't** have a class, the shape varies, you only need a few fields, or you need to modify JSON and re-serialize it.

---

## Comparison at a Glance

| | `JsonDocument` | `JsonNode` |
|---|---|---|
| **Mutability** | Read-only | Mutable (read + write) |
| **Available since** | .NET Core 3.0 | .NET 6 |
| **Memory model** | Pooled (`ArrayPool`), **must dispose** | GC-managed, no disposal needed |
| **Central type** | `JsonElement` (struct) | `JsonNode` (class hierarchy) |
| **Best for** | Parse, read a few values, discard | Parse, modify, re-serialize |
| **Performance** | Faster for read-only access | Slightly more overhead due to object graph |
| **Null root** | `doc.RootElement` always valid | `JsonNode.Parse()` returns `JsonNode?` (nullable) |

---

## JsonDocument (Read-Only DOM)

`JsonDocument` parses a JSON string (or bytes or stream) into a read-only tree of **`JsonElement`** values. It is optimized for scenarios where you parse JSON, extract what you need, and move on.

### Parsing

```csharp
using System.Text.Json;

string jsonString = """
    {
        "Name": "Long",
        "Age": 28,
        "Address": {
            "City": "Chicago",
            "Zip": "60601"
        },
        "Tags": ["dev", "csharp", "dotnet"]
    }
    """;

// Parse returns a JsonDocument -- always wrap in using
using JsonDocument doc = JsonDocument.Parse(jsonString);
JsonElement root = doc.RootElement;
```

You can also parse from `ReadOnlyMemory<byte>`, `Stream`, or `ReadOnlySequence<byte>`:

```csharp
// From a stream (async)
using JsonDocument doc = await JsonDocument.ParseAsync(stream);

// From UTF-8 bytes
byte[] utf8Bytes = Encoding.UTF8.GetBytes(jsonString);
using JsonDocument doc = JsonDocument.Parse(utf8Bytes);
```

> [!ad-important] JsonDocument is IDisposable
> `JsonDocument` rents memory from `ArrayPool<byte>`. You **must** dispose it (use a `using` statement). If you need a `JsonElement` to outlive the document, call `element.Clone()` -- this copies the data to GC-managed memory.

```csharp
JsonElement cloned;
using (JsonDocument doc = JsonDocument.Parse(jsonString))
{
    // Clone before the document is disposed
    cloned = doc.RootElement.GetProperty("Name").Clone();
}
// cloned is still valid here
Console.WriteLine(cloned.GetString()); // "Long"
```

### Navigating with JsonElement

`JsonElement` is the core type. It represents any JSON value (object, array, string, number, bool, null).

```csharp
using JsonDocument doc = JsonDocument.Parse(jsonString);
JsonElement root = doc.RootElement;

// Reading simple properties
string name = root.GetProperty("Name").GetString()!;   // "Long"
int age = root.GetProperty("Age").GetInt32();            // 28

// Nested objects -- chain GetProperty calls
string city = root.GetProperty("Address")
                  .GetProperty("City")
                  .GetString()!;                          // "Chicago"

// Arrays -- iterate with EnumerateArray()
foreach (JsonElement tag in root.GetProperty("Tags").EnumerateArray())
{
    Console.WriteLine(tag.GetString());
    // "dev", "csharp", "dotnet"
}

// Objects -- iterate all properties with EnumerateObject()
foreach (JsonProperty prop in root.EnumerateObject())
{
    Console.WriteLine($"{prop.Name}: {prop.Value.ValueKind}");
    // Name: String, Age: Number, Address: Object, Tags: Array
}
```

### Safe Navigation with TryGetProperty

`GetProperty` throws `KeyNotFoundException` if the property doesn't exist. For optional or uncertain fields, use `TryGetProperty`:

```csharp
// Safe -- returns false if the property doesn't exist
if (root.TryGetProperty("Email", out JsonElement email))
{
    Console.WriteLine(email.GetString());
}
else
{
    Console.WriteLine("No email found");
}

// Also safe -- for numeric values
if (root.GetProperty("Age").TryGetInt32(out int parsedAge))
{
    Console.WriteLine($"Age: {parsedAge}");
}
```

### JsonElement.ValueKind

Every `JsonElement` has a `ValueKind` property that tells you what kind of JSON value it holds:

| `JsonValueKind` | Meaning | Example JSON |
|---|---|---|
| `Object` | JSON object | `{ "a": 1 }` |
| `Array` | JSON array | `[1, 2, 3]` |
| `String` | JSON string | `"hello"` |
| `Number` | JSON number | `42`, `3.14` |
| `True` | JSON `true` | `true` |
| `False` | JSON `false` | `false` |
| `Null` | JSON `null` | `null` |
| `Undefined` | No value / uninitialized | (default `JsonElement`) |

```csharp
JsonElement age = root.GetProperty("Age");

switch (age.ValueKind)
{
    case JsonValueKind.Number:
        Console.WriteLine($"It's a number: {age.GetInt32()}");
        break;
    case JsonValueKind.String:
        Console.WriteLine($"It's a string: {age.GetString()}");
        break;
    case JsonValueKind.Null:
        Console.WriteLine("It's null");
        break;
}
```

> [!ad-note] Checking for null
> Always check `element.ValueKind == JsonValueKind.Null` before calling `GetString()`, `GetInt32()`, etc. Calling `GetString()` on a null element returns `null`, but calling `GetInt32()` on a null element throws an `InvalidOperationException`.

### Writing JsonElement Back to JSON

```csharp
using JsonDocument doc = JsonDocument.Parse(jsonString);
JsonElement address = doc.RootElement.GetProperty("Address");

// Write a single element to a string
using var stream = new MemoryStream();
using var writer = new Utf8JsonWriter(stream, new JsonWriterOptions { Indented = true });
address.WriteTo(writer);
writer.Flush();

string addressJson = Encoding.UTF8.GetString(stream.ToArray());
// { "City": "Chicago", "Zip": "60601" }
```

---

## JsonNode (Mutable DOM, .NET 6+)

`JsonNode` is a mutable DOM -- you can parse JSON, modify it, add/remove properties, and re-serialize. It lives in the `System.Text.Json.Nodes` namespace.

The `JsonNode` hierarchy:

| Type | Represents | Example |
|---|---|---|
| `JsonObject` | JSON object `{ }` | Properties with string keys |
| `JsonArray` | JSON array `[ ]` | Ordered list of values |
| `JsonValue` | Primitive value | String, number, bool, null |

`JsonNode` is the abstract base class; you work with the concrete types above.

### Parsing

```csharp
using System.Text.Json.Nodes;

string jsonString = """
    {
        "Name": "Long",
        "Age": 28,
        "Tags": ["dev", "csharp"]
    }
    """;

// Parse returns JsonNode? (nullable -- null if the JSON is "null")
JsonNode? root = JsonNode.Parse(jsonString);
```

### Reading Values

```csharp
// Use the indexer and GetValue<T>()
string name = root!["Name"]!.GetValue<string>();   // "Long"
int age = root["Age"]!.GetValue<int>();              // 28

// Arrays -- index into them
string firstTag = root["Tags"]![0]!.GetValue<string>();  // "dev"

// Iterate an array
JsonArray tags = root["Tags"]!.AsArray();
foreach (JsonNode? tag in tags)
{
    Console.WriteLine(tag!.GetValue<string>());
}
```

> [!ad-warning] Null reference danger
> `root["PropertyName"]` returns `JsonNode?`. If the property doesn't exist, it returns `null` (not an exception). Calling `.GetValue<T>()` on `null` will throw `NullReferenceException`. Always use null-conditional operators or null checks:
> ```csharp
> // Safe access
> string? email = root!["Email"]?.GetValue<string>();
> 
> // Or check explicitly
> if (root!["Email"] is JsonNode emailNode)
> {
>     string email = emailNode.GetValue<string>();
> }
> ```

### Modifying Values

This is the key advantage over `JsonDocument` -- you can change the JSON in place:

```csharp
JsonNode? root = JsonNode.Parse(jsonString);

// Change an existing value
root!["Age"] = 29;

// Add a new property
root["Email"] = "long@example.com";

// Remove a property
root.AsObject().Remove("Tags");

// Add to an array
root["Skills"] = new JsonArray("C#", "SQL");
root["Skills"]!.AsArray().Add("Docker");

// The JSON now reflects all changes
string updated = root.ToJsonString(new JsonSerializerOptions { WriteIndented = true });
```

### Building JSON from Scratch

You don't have to parse existing JSON -- you can construct a `JsonNode` tree programmatically:

```csharp
var node = new JsonObject
{
    ["Name"] = "Long",
    ["Age"] = 28,
    ["IsActive"] = true,
    ["Address"] = new JsonObject
    {
        ["City"] = "Chicago",
        ["Zip"] = "60601"
    },
    ["Tags"] = new JsonArray("dev", "csharp", "dotnet"),
    ["Score"] = 95.5
};

// Serialize to a JSON string
string json = node.ToJsonString(new JsonSerializerOptions { WriteIndented = true });
```

Output:

```json
{
  "Name": "Long",
  "Age": 28,
  "IsActive": true,
  "Address": {
    "City": "Chicago",
    "Zip": "60601"
  },
  "Tags": [
    "dev",
    "csharp",
    "dotnet"
  ],
  "Score": 95.5
}
```

### Re-serializing with Options

`ToJsonString()` accepts `JsonSerializerOptions` for formatting control:

```csharp
var options = new JsonSerializerOptions
{
    WriteIndented = true,
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    Encoder = System.Text.Encodings.Web.JavaScriptEncoder.UnsafeRelaxedJsonEscaping
};

string json = node.ToJsonString(options);
```

> [!ad-note] PropertyNamingPolicy and JsonNode
> `PropertyNamingPolicy` does **not** rename keys that you explicitly set on a `JsonObject`. It only affects property names during `JsonSerializer.Serialize<T>()` with a typed object. If you build a `JsonObject` with key `"Name"`, it stays `"Name"` regardless of the naming policy.

---

## Converting Between DOM and Typed Objects

You can bridge between the DOM APIs and strongly-typed serialization:

### JsonElement to T

```csharp
using JsonDocument doc = JsonDocument.Parse(jsonString);
JsonElement addressElement = doc.RootElement.GetProperty("Address");

// Deserialize a JsonElement to a typed object
Address? address = addressElement.Deserialize<Address>();
```

### JsonNode to T

```csharp
JsonNode? root = JsonNode.Parse(jsonString);
JsonNode? addressNode = root!["Address"];

// Deserialize a JsonNode to a typed object
Address? address = addressNode.Deserialize<Address>();
```

### T to JsonNode

```csharp
var person = new Person { Name = "Long", Age = 28 };

// Serialize a typed object to a JsonNode
JsonNode? node = JsonSerializer.SerializeToNode(person);
```

### T to JsonElement

```csharp
var person = new Person { Name = "Long", Age = 28 };

// Serialize to a JsonElement via JsonDocument
JsonElement element = JsonSerializer.SerializeToElement(person);
```

---

## Choosing the Right Approach

| Scenario | Use |
|---|---|
| Known JSON structure, need all data | `JsonSerializer.Deserialize<T>()` |
| Unknown structure, read a few fields, discard | `JsonDocument` |
| Need to patch/modify JSON and re-serialize | `JsonNode` |
| Build JSON dynamically (no source class) | `JsonNode` (`JsonObject` / `JsonArray`) |
| Stream very large JSON, process element-by-element | `Utf8JsonReader` directly |
| Extract one section and deserialize it | `JsonDocument` + `element.Deserialize<T>()` |

> [!ad-important] Performance consideration
> For read-only access to a small number of fields, `JsonDocument` is faster than `JsonNode` because it avoids creating a full object graph. However, `JsonDocument` requires disposal and doesn't support modification. If you need to modify even one property, `JsonNode` is the right choice.

---

## See Also

- [[Custom JsonConverters]] -- writing custom serialization logic for specific types
- [[XML Serialization]] -- XML-based serialization with `XmlSerializer`
- [[BinaryFormatter and Modern Alternatives]] -- binary serialization options
