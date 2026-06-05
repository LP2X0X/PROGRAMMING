---
tags:
 - csharp
 - serialization
 - json
---

## JsonSerializer -- The High-Level Serialize/Deserialize API

`System.Text.Json.JsonSerializer` is a **static class** that provides the primary high-level API for converting between .NET objects and JSON. It is the entry point you will use in the vast majority of serialization scenarios. Under the hood, it uses `Utf8JsonReader` and `Utf8JsonWriter` to do the actual byte-level work, but it abstracts away all that complexity.

```csharp
using System.Text.Json;
```

This note covers every overload you need to know, what gets serialized by default, how deserialization constructs objects, error handling, and the subtleties that trip people up.

---

## Serializing Objects

### To a `string`

The most common overload. Produces a UTF-16 `string` containing JSON.

```csharp
var person = new Person { Name = "Long", Age = 28 };

// Basic -- produces compact JSON
string json = JsonSerializer.Serialize(person);
// Result: {"Name":"Long","Age":28}

// Pretty-printed -- useful for logging and debugging
string pretty = JsonSerializer.Serialize(person, new JsonSerializerOptions
{
    WriteIndented = true
});
// Result:
// {
//   "Name": "Long",
//   "Age": 28
// }
```

### To UTF-8 Bytes

Faster than `Serialize<T>()` because JSON is natively UTF-8. This avoids the internal UTF-8-to-UTF-16 transcoding that the `string` overload must perform.

```csharp
byte[] utf8Bytes = JsonSerializer.SerializeToUtf8Bytes(person);
// Use this when writing to a file, HTTP response, or anywhere that accepts bytes
```

> [!ad-info] Performance: Prefer Bytes When Possible
> If your destination is a `Stream`, an HTTP response body, or a byte buffer, use `SerializeToUtf8Bytes` or the `Stream` overload. The `string` overload does extra work: it first writes UTF-8 internally, then transcodes to UTF-16 for the `string`. If you are just going to convert that string back to bytes anyway (e.g., `Encoding.UTF8.GetBytes(json)`), you are paying for two conversions instead of zero.

### To a Stream (Synchronous)

Writes JSON directly to any `Stream` -- a `FileStream`, `MemoryStream`, `HttpResponseStream`, etc.

```csharp
using var stream = File.Create("person.json");
JsonSerializer.Serialize(stream, person);
// JSON is written directly to the file, no intermediate string
```

### To a Stream (Asynchronous)

The async overload is critical for I/O-bound scenarios (file I/O, network) to avoid blocking threads:

```csharp
using var stream = File.Create("person.json");
await JsonSerializer.SerializeAsync(stream, person);

// With options and cancellation
await JsonSerializer.SerializeAsync(stream, person, options, cancellationToken);
```

> [!ad-important] Always Prefer Async for I/O Streams
> When writing to a `FileStream` or `NetworkStream`, use `SerializeAsync`. The synchronous `Serialize` to a `Stream` will block the calling thread while waiting for the I/O operation, which is wasteful in async contexts like ASP.NET Core.

### With Runtime Type

When the type is not known at compile time:

```csharp
object personObj = new Person { Name = "Long", Age = 28 };

// Using generic overload with object -- serializes ONLY properties on object (none!)
string bad = JsonSerializer.Serialize(personObj);
// Result: {} -- NOT what you want!

// Using Type parameter -- serializes properties on the ACTUAL runtime type
string good = JsonSerializer.Serialize(personObj, personObj.GetType());
// Result: {"Name":"Long","Age":28}
```

> [!ad-warning] The `Serialize<object>()` Trap
> If you call `JsonSerializer.Serialize<object>(myPerson)`, the serializer sees `T = object` and serializes only properties declared on `System.Object` -- which has none. You get `{}`. This is one of the most common mistakes. To serialize the actual type, either use the non-generic overload with `GetType()` or cast to the correct type: `JsonSerializer.Serialize((Person)personObj)`.
>
> Starting in .NET 7, you can also pass `JsonSerializerOptions` with `TypeInfoResolver` to handle polymorphic serialization more elegantly. See [[Serialization Attributes]] for `[JsonDerivedType]`.

---

## Deserializing Objects

### From a `string`

```csharp
string json = """{"Name":"Long","Age":28}""";

// Generic overload -- returns T?
Person? person = JsonSerializer.Deserialize<Person>(json);

// Non-generic overload -- returns object?
object? obj = JsonSerializer.Deserialize(json, typeof(Person));
```

### From UTF-8 Bytes (`ReadOnlySpan<byte>`)

```csharp
byte[] utf8Bytes = File.ReadAllBytes("person.json");
Person? person = JsonSerializer.Deserialize<Person>(utf8Bytes);

// Or from a ReadOnlySpan<byte>
ReadOnlySpan<byte> span = utf8Bytes.AsSpan();
Person? person2 = JsonSerializer.Deserialize<Person>(span);
```

### From a Stream (Synchronous and Asynchronous)

```csharp
// Synchronous
using var readStream = File.OpenRead("person.json");
Person? person = JsonSerializer.Deserialize<Person>(readStream);

// Asynchronous -- preferred for I/O streams
using var asyncStream = File.OpenRead("person.json");
Person? person2 = await JsonSerializer.DeserializeAsync<Person>(asyncStream);

// With cancellation token
Person? person3 = await JsonSerializer.DeserializeAsync<Person>(
    asyncStream, 
    options, 
    cancellationToken
);
```

### From a `JsonElement` or `JsonDocument`

When you have already parsed JSON into the DOM:

```csharp
using JsonDocument doc = JsonDocument.Parse(json);
JsonElement element = doc.RootElement;

// Deserialize a JsonElement into a typed object
Person? person = element.Deserialize<Person>();
```

### From a `JsonNode`

```csharp
JsonNode? node = JsonNode.Parse(json);
Person? person = node.Deserialize<Person>();
```

---

## Serializing and Deserializing Collections

`JsonSerializer` handles all standard .NET collections:

```csharp
// ---- Lists ----
var people = new List<Person>
{
    new() { Name = "Long", Age = 28 },
    new() { Name = "Alice", Age = 30 }
};

string json = JsonSerializer.Serialize(people);
// [{"Name":"Long","Age":28},{"Name":"Alice","Age":30}]

List<Person>? restored = JsonSerializer.Deserialize<List<Person>>(json);

// ---- Arrays ----
Person[] array = JsonSerializer.Deserialize<Person[]>(json)!;

// ---- Dictionaries ----
var scores = new Dictionary<string, int>
{
    ["Alice"] = 95,
    ["Bob"] = 87
};

string dictJson = JsonSerializer.Serialize(scores);
// {"Alice":95,"Bob":87}

Dictionary<string, int>? restoredDict = 
    JsonSerializer.Deserialize<Dictionary<string, int>>(dictJson);

// ---- Nested collections ----
var nested = new Dictionary<string, List<int>>
{
    ["primes"] = new() { 2, 3, 5, 7 },
    ["fibonacci"] = new() { 1, 1, 2, 3, 5 }
};

string nestedJson = JsonSerializer.Serialize(nested);
// {"primes":[2,3,5,7],"fibonacci":[1,1,2,3,5]}
```

> [!ad-note] Dictionary Key Constraints
> Dictionary keys must be `string`, `int`, `long`, `Guid`, `Enum`, or other types that have a built-in converter that can produce a JSON property name. If you need a complex type as a key (e.g., a tuple), you'll need a custom converter.

---

## What Gets Serialized by Default

Understanding the default behavior is critical. Many bugs come from incorrect assumptions about what `JsonSerializer` includes or excludes.

| Member Type | Serialized? | Deserialized? | Notes |
|---|---|---|---|
| Public property with `get` and `set` | Yes | Yes | The standard case |
| Public property with `get` only (read-only) | Yes | **No** | Value is written to JSON but ignored on read |
| Public property with `set` only (write-only) | **No** | Yes | Unusual but possible |
| Public field | **No** | **No** | Opt-in with `JsonSerializerOptions.IncludeFields = true` or `[JsonInclude]` |
| Private property | **No** | **No** | Not accessible by default |
| Private field | **No** | **No** | Not accessible by default |
| `internal` property | **No** | **No** | Not public = not serialized |
| `protected` property | **No** | **No** | Not public = not serialized |
| Static property | **No** | **No** | Instance members only |
| Indexer (`this[]`) | **No** | **No** | Not supported |
| `init`-only property | Yes | Yes | Treated like a setter during deserialization |
| `required` property (.NET 7+) | Yes | Yes (required in JSON) | `JsonSerializer` respects `required` |

```csharp
public class Demo
{
    public string Included { get; set; }        // Serialized + deserialized
    public string ReadOnly { get; }             // Serialized only (not deserialized)
    public int PublicField;                     // NOT serialized (unless IncludeFields = true)
    private string Hidden { get; set; }         // NOT serialized
    internal string AlsoHidden { get; set; }    // NOT serialized
    
    [JsonInclude]
    public int OptedInField;                    // Serialized (explicitly opted in)
}
```

> [!ad-warning] Read-Only Properties Are Silently Skipped on Deserialization
> If you have a property like `public string Name { get; }` (no setter), `JsonSerializer` will serialize it but silently ignore it during deserialization. The property will have its default value. This is a common source of confusion. To deserialize into read-only properties, use a parameterized constructor with `[JsonConstructor]`. See [[Serialization Attributes]].

---

## How Deserialization Constructs Objects

`JsonSerializer` needs to create an instance of your type before it can populate properties. Here is the precedence:

1. **Parameterless constructor** -- the default. If your class has a public parameterless constructor, the serializer calls it, then sets each property via its public setter.

2. **Parameterized constructor with `[JsonConstructor]`** -- if you mark a constructor, the serializer matches JSON property names to constructor parameter names (case-insensitive) and passes values through the constructor.

3. **Single parameterized constructor** (.NET 6+) -- if there is exactly one public constructor and it has parameters, the serializer uses it automatically without `[JsonConstructor]`.

```csharp
// Approach 1: Parameterless constructor (default)
public class PersonV1
{
    public string Name { get; set; } = "";
    public int Age { get; set; }
    // Implicit parameterless constructor -- works out of the box
}

// Approach 2: Parameterized constructor with [JsonConstructor]
public class PersonV2
{
    public string Name { get; }
    public int Age { get; }
    
    [JsonConstructor]
    public PersonV2(string name, int age)   // parameter names must match property names (case-insensitive)
    {
        Name = name;
        Age = age;
    }
}

// Approach 3: Records -- work naturally because they have a primary constructor
public record PersonV3(string Name, int Age);
// JsonSerializer maps "Name" and "Age" from JSON to the constructor parameters automatically
```

> [!ad-note] Constructor Parameter Name Matching
> The serializer matches constructor parameter names to JSON property names **case-insensitively**. The parameter `name` matches JSON property `"Name"`. If you have renamed a property with `[JsonPropertyName("full_name")]`, the constructor parameter must also be `full_name` (or you need a `[JsonPropertyName]` on the parameter -- not supported directly, so use a different approach).

> [!ad-warning] No Parameterless Constructor + No `[JsonConstructor]` + Multiple Constructors = Error
> If your class has no parameterless constructor and multiple parameterized constructors without `[JsonConstructor]`, deserialization will throw a `NotSupportedException`. The serializer does not know which constructor to use. Always annotate with `[JsonConstructor]` when you have multiple constructors.

---

## Handling Null Values

```csharp
public class Person
{
    public string Name { get; set; } = "";
    public string? MiddleName { get; set; }    // nullable
    public int Age { get; set; }
}

var person = new Person { Name = "Long", MiddleName = null, Age = 28 };

// Default behavior: null values ARE included
string json = JsonSerializer.Serialize(person);
// {"Name":"Long","MiddleName":null,"Age":28}

// Exclude null values
var options = new JsonSerializerOptions
{
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull
};
string noNulls = JsonSerializer.Serialize(person, options);
// {"Name":"Long","Age":28}
```

See [[JsonSerializerOptions]] for all `DefaultIgnoreCondition` values and [[Serialization Attributes]] for per-property control with `[JsonIgnore]`.

---

## Enums

By default, enums serialize as **integers**:

```csharp
public enum Color { Red, Green, Blue }

public class Widget
{
    public string Label { get; set; } = "";
    public Color Color { get; set; }
}

var widget = new Widget { Label = "button", Color = Color.Green };
string json = JsonSerializer.Serialize(widget);
// {"Label":"button","Color":1}   <-- Green = 1
```

To serialize enums as **string names**:

```csharp
var options = new JsonSerializerOptions
{
    Converters = { new JsonStringEnumConverter() }  // global for all enums
};

string json = JsonSerializer.Serialize(widget, options);
// {"Label":"button","Color":"Green"}
```

Or per-property using [[Serialization Attributes]]:

```csharp
public class Widget
{
    public string Label { get; set; } = "";
    
    [JsonConverter(typeof(JsonStringEnumConverter))]
    public Color Color { get; set; }
}
```

> [!ad-note] JsonStringEnumConverter Naming Policy
> `JsonStringEnumConverter` accepts an optional `JsonNamingPolicy` parameter:
> ```csharp
> // "green" instead of "Green"
> new JsonStringEnumConverter(JsonNamingPolicy.CamelCase)
> ```
> In .NET 8+, there is also `JsonStringEnumConverter<TEnum>` for source-generated scenarios.

---

## Error Handling

`JsonSerializer` throws `JsonException` when it encounters invalid JSON or a type mismatch:

```csharp
try
{
    // Invalid JSON -- missing closing brace
    Person? p = JsonSerializer.Deserialize<Person>("""{"Name":"Long" """);
}
catch (JsonException ex)
{
    Console.WriteLine(ex.Message);
    // 'L' is an invalid end of a number. Expected a delimiter. Path: $ | LineNumber: 0 | ...
    
    Console.WriteLine(ex.Path);        // "$" -- JSON path where error occurred
    Console.WriteLine(ex.LineNumber);   // 0
    Console.WriteLine(ex.BytePositionInLine);  // byte offset in the line
}
```

Common causes of `JsonException`:

| Cause | Example |
|---|---|
| Malformed JSON | Missing quotes, braces, brackets, trailing commas (when not allowed) |
| Type mismatch | JSON has `"hello"` but C# property is `int` |
| Unknown properties | Only when `UnmappedMemberHandling = Disallow` (.NET 8+) |
| Max depth exceeded | JSON nesting deeper than `MaxDepth` (default 64) |
| Circular reference | Without `ReferenceHandler` configured |

```csharp
// Defensive deserialization pattern
public static T? SafeDeserialize<T>(string json, JsonSerializerOptions? options = null)
{
    try
    {
        return JsonSerializer.Deserialize<T>(json, options);
    }
    catch (JsonException ex)
    {
        // Log the error with the path for debugging
        Console.Error.WriteLine($"JSON deserialization failed at {ex.Path}: {ex.Message}");
        return default;
    }
}
```

> [!ad-warning] NotSupportedException from JsonSerializer
> A `NotSupportedException` (not `JsonException`) is thrown when the serializer encounters a type it fundamentally cannot handle -- for example, a type with no accessible constructor, a `Span<T>` property, or a type that has no registered converter. This is a code-level problem, not a data-level problem.

---

## Streaming Deserialization -- `DeserializeAsyncEnumerable` (.NET 6+)

For very large JSON arrays, you can deserialize elements one at a time without loading the entire array into memory:

```csharp
using var stream = File.OpenRead("large-array.json");  // Contains: [{"Name":"A",...}, {"Name":"B",...}, ...]

await foreach (Person? person in JsonSerializer.DeserializeAsyncEnumerable<Person>(stream))
{
    if (person is not null)
    {
        Console.WriteLine(person.Name);
    }
}
// Only one Person object is in memory at a time -- not the entire array
```

> [!ad-info] When to Use Streaming Deserialization
> Use `DeserializeAsyncEnumerable` when:
> - The JSON file/stream contains a large array (thousands or millions of elements)
> - You process elements one-by-one and don't need the entire collection in memory
> - Memory pressure is a concern
>
> The JSON **must** be a root-level array `[...]`. This does not work for JSON objects or nested arrays.

---

## Complete Example -- Real-World Round-Trip

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;

// ---- Define the model ----
public class Employee
{
    public required string Name { get; init; }
    public required string Department { get; init; }
    public decimal Salary { get; init; }
    public DateOnly HireDate { get; init; }
    
    [JsonConverter(typeof(JsonStringEnumConverter))]
    public EmployeeLevel Level { get; init; }
    
    [JsonIgnore]   // never serialize this
    public string InternalNotes { get; set; } = "";
}

public enum EmployeeLevel { Junior, Mid, Senior, Lead, Principal }

// ---- Configure once, reuse everywhere ----
var options = new JsonSerializerOptions
{
    WriteIndented = true,
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull
};

// ---- Serialize ----
var employees = new List<Employee>
{
    new()
    {
        Name = "Long Pham",
        Department = "Engineering",
        Salary = 95_000m,
        HireDate = new DateOnly(2023, 6, 15),
        Level = EmployeeLevel.Mid,
        InternalNotes = "This will not appear in JSON"
    }
};

// Write to file asynchronously
using (var writeStream = File.Create("employees.json"))
{
    await JsonSerializer.SerializeAsync(writeStream, employees, options);
}

// ---- Deserialize ----
using (var readStream = File.OpenRead("employees.json"))
{
    var restored = await JsonSerializer.DeserializeAsync<List<Employee>>(readStream, options);
    
    foreach (var emp in restored ?? [])
    {
        Console.WriteLine($"{emp.Name} ({emp.Level}) - {emp.Department}");
    }
}
```

Output file `employees.json`:
```json
[
  {
    "name": "Long Pham",
    "department": "Engineering",
    "salary": 95000,
    "hireDate": "2023-06-15",
    "level": "Mid"
  }
]
```

Notice:
- `InternalNotes` is absent (excluded by `[JsonIgnore]`)
- Property names are camelCase (from `JsonNamingPolicy.CamelCase`)
- `Level` is a string `"Mid"` (from `JsonStringEnumConverter`)
- `HireDate` is ISO 8601 format (built-in `DateOnly` support)

---

## API Quick Reference

| Method | Input | Output | Notes |
|---|---|---|---|
| `Serialize<T>(T)` | Object | `string` | Most common |
| `Serialize(Stream, T)` | Object | Writes to stream | Sync |
| `SerializeAsync(Stream, T)` | Object | Writes to stream | Async, preferred for I/O |
| `SerializeToUtf8Bytes<T>(T)` | Object | `byte[]` | Fastest -- no UTF-16 conversion |
| `Deserialize<T>(string)` | JSON string | `T?` | Most common |
| `Deserialize<T>(ReadOnlySpan<byte>)` | UTF-8 bytes | `T?` | Fast -- no string allocation |
| `Deserialize<T>(Stream)` | Stream | `T?` | Sync |
| `DeserializeAsync<T>(Stream)` | Stream | `T?` | Async, preferred |
| `DeserializeAsyncEnumerable<T>(Stream)` | Stream | `IAsyncEnumerable<T?>` | Streaming -- one item at a time (.NET 6+) |

All overloads accept an optional `JsonSerializerOptions` parameter. See [[JsonSerializerOptions]] for all configuration options.

---

## Related Notes

- [[Object Serialization Overview]] -- What serialization is, why it matters, format comparison
- [[System.Text.Json Overview]] -- Architecture, API levels, comparison with Newtonsoft.Json
- [[JsonSerializerOptions]] -- Configure naming policies, null handling, converters, circular references
- [[Serialization Attributes]] -- Per-property control: `[JsonPropertyName]`, `[JsonIgnore]`, `[JsonConstructor]`, etc.
