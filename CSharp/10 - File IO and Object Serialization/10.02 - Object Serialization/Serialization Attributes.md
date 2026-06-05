---
tags:
 - csharp
 - serialization
 - json
---

## Serialization Attributes -- Per-Property and Per-Type Control

While [[JsonSerializerOptions]] configures serialization **globally**, attributes let you control serialization behavior on a **per-property** or **per-type** basis. They live in the `System.Text.Json.Serialization` namespace and are applied directly to your model classes.

```csharp
using System.Text.Json.Serialization;
```

Attributes always take precedence over global options. For example, `[JsonPropertyName("my_name")]` overrides whatever `PropertyNamingPolicy` is set in `JsonSerializerOptions`.

---

## Complete Attribute Reference

| Attribute | Target | .NET Version | Purpose |
|---|---|---|---|
| `[JsonPropertyName]` | Property / Field | Core 3.0+ | Rename the property in JSON |
| `[JsonIgnore]` | Property / Field | Core 3.0+ | Exclude from serialization/deserialization |
| `[JsonInclude]` | Property / Field | .NET 5+ | Include a public field or non-public accessor |
| `[JsonRequired]` | Property / Field | .NET 7+ | Property must be present in JSON during deserialization |
| `[JsonPropertyOrder]` | Property / Field | .NET 6+ | Control the order of properties in serialized JSON |
| `[JsonNumberHandling]` | Property / Class | .NET 5+ | Per-property/type number handling |
| `[JsonConverter]` | Property / Class | Core 3.0+ | Apply a custom converter |
| `[JsonConstructor]` | Constructor | Core 3.0+ | Specify which constructor to use for deserialization |
| `[JsonDerivedType]` | Class | .NET 7+ | Register derived types for polymorphic serialization |
| `[JsonPolymorphic]` | Class | .NET 7+ | Configure polymorphic serialization behavior |
| `[JsonExtensionData]` | Property | Core 3.0+ | Capture unmapped JSON properties |
| `[JsonSourceGenerationOptions]` | Class | .NET 6+ | Configure source-generated serialization |
| `[JsonSerializable]` | Class | .NET 6+ | Register a type for source generation |
| `[JsonUnmappedMemberHandling]` | Class | .NET 8+ | Per-type handling of unknown JSON properties |
| `[JsonObjectCreationHandling]` | Property / Class | .NET 8+ | Populate vs. replace during deserialization |

---

## `[JsonPropertyName]` -- Rename a Property in JSON

Maps a C# property name to a specific JSON property name. This is the most commonly used attribute.

```csharp
public class Person
{
    [JsonPropertyName("full_name")]
    public string FullName { get; set; } = "";
    
    [JsonPropertyName("date_of_birth")]
    public DateOnly DateOfBirth { get; set; }
    
    // No attribute -- uses the naming policy (or PascalCase by default)
    public int Age { get; set; }
}

// Serialized with CamelCase naming policy:
// {
//   "full_name": "Long Pham",        <-- [JsonPropertyName] wins over naming policy
//   "date_of_birth": "1998-01-15",   <-- [JsonPropertyName] wins over naming policy
//   "age": 28                         <-- naming policy applies (no attribute)
// }
```

> [!ad-important] `[JsonPropertyName]` Always Wins
> When a property has `[JsonPropertyName]`, that exact name is used in JSON -- the `PropertyNamingPolicy` in [[JsonSerializerOptions]] is ignored for that property. This is how you handle APIs with inconsistent naming (some properties camelCase, others snake_case).

> [!ad-note] Deserialization Also Uses `[JsonPropertyName]`
> The attribute works in both directions. During deserialization, the serializer looks for `"full_name"` in the JSON to populate `FullName`. The original C# name `"FullName"` will **not** match unless `PropertyNameCaseInsensitive` is true and there happens to be no `"full_name"` key.

---

## `[JsonIgnore]` -- Exclude a Property

Prevents a property from appearing in serialized JSON and/or from being read during deserialization.

```csharp
public class User
{
    public string Username { get; set; } = "";
    
    [JsonIgnore]   // always excluded -- both serialization and deserialization
    public string PasswordHash { get; set; } = "";
    
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingNull)]
    public string? MiddleName { get; set; }
    
    [JsonIgnore(Condition = JsonIgnoreCondition.WhenWritingDefault)]
    public int LoginCount { get; set; }    // omitted when 0
    
    [JsonIgnore(Condition = JsonIgnoreCondition.Always)]
    public string InternalId { get; set; } = "";   // same as no Condition
}
```

**`JsonIgnoreCondition` Values:**

| Value | Serialization Behavior | Deserialization Behavior |
|---|---|---|
| `Always` (default) | Never written | Never read |
| `Never` | Always written | Always read (overrides global `DefaultIgnoreCondition`) |
| `WhenWritingNull` | Omit when `null` | Still read from JSON |
| `WhenWritingDefault` | Omit when equal to type default (`null`, `0`, `false`, etc.) | Still read from JSON |

> [!ad-info] `[JsonIgnore(Condition = JsonIgnoreCondition.Never)]` -- Overriding Global Settings
> If you have set `DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull` globally in [[JsonSerializerOptions]], but a specific property should always be included even when `null`, use:
> ```csharp
> [JsonIgnore(Condition = JsonIgnoreCondition.Never)]
> public string? AlwaysIncluded { get; set; }
> ```
> This is the per-property override for the global setting.

---

## `[JsonInclude]` -- Opt In Public Fields or Non-Public Accessors

By default, `JsonSerializer` only serializes **public properties with public getters**. `[JsonInclude]` explicitly opts in members that would otherwise be excluded.

```csharp
public class Measurement
{
    // Public field -- normally excluded, [JsonInclude] opts it in
    [JsonInclude]
    public double Value;
    
    // Property with private setter -- the setter is normally inaccessible to the serializer
    // [JsonInclude] allows deserialization through the private setter
    [JsonInclude]
    public string Unit { get; private set; } = "";
    
    // Property with internal getter -- normally excluded
    [JsonInclude]
    public DateTime Timestamp { internal get; set; }
}
```

> [!ad-note] You Don't Need `[JsonInclude]` for `init` Setters
> Properties with `init` accessors are already serializable and deserializable by default. `[JsonInclude]` is only needed for truly private/internal accessors and public fields.

> [!ad-warning] `[JsonInclude]` on Private Members
> `[JsonInclude]` does NOT work on fully private properties or private fields. The member must be **public** but have a non-public accessor (like `public string Name { get; private set; }`). For serializing truly private state, you need a custom `JsonConverter<T>`.

---

## `[JsonRequired]` (.NET 7+) -- Mandatory Properties

Ensures a property is present in the JSON during deserialization. If the property is missing, deserialization throws a `JsonException`.

```csharp
public class CreateUserRequest
{
    [JsonRequired]
    public string Username { get; set; } = "";
    
    [JsonRequired]
    public string Email { get; set; } = "";
    
    public string? DisplayName { get; set; }   // optional
}

// This works:
var json1 = """{"Username":"long","Email":"long@example.com"}""";
var user1 = JsonSerializer.Deserialize<CreateUserRequest>(json1);  // OK

// This throws JsonException: "JSON deserialization for type requires the 'Username' property"
var json2 = """{"Email":"long@example.com"}""";
var user2 = JsonSerializer.Deserialize<CreateUserRequest>(json2);  // THROWS
```

> [!ad-info] `[JsonRequired]` vs C# `required` Keyword
> In .NET 7+, `System.Text.Json` respects **both** the `[JsonRequired]` attribute and the C# `required` modifier. They do the same thing from the serializer's perspective -- both cause a `JsonException` if the property is missing during deserialization:
> ```csharp
> // These are equivalent for JsonSerializer:
> [JsonRequired]
> public string Name { get; set; }
> 
> // C# 11 required keyword -- also enforced by JsonSerializer in .NET 7+
> public required string Name { get; set; }
> ```
> The difference is that `required` is also enforced by the **compiler** at the call site (you must set it when constructing the object), while `[JsonRequired]` is only enforced during deserialization.

---

## `[JsonPropertyOrder]` (.NET 6+) -- Control Property Order

By default, properties are serialized in the order they are declared in the class. `[JsonPropertyOrder]` overrides this with explicit ordering.

```csharp
public class ApiResponse
{
    [JsonPropertyOrder(3)]
    public object? Data { get; set; }
    
    [JsonPropertyOrder(1)]           // appears first
    public bool Success { get; set; }
    
    [JsonPropertyOrder(2)]
    public string? Message { get; set; }
}

// Serialized as:
// {"Success":true,"Message":"OK","Data":{...}}
// NOT the declaration order: Data, Success, Message
```

Properties without `[JsonPropertyOrder]` have a default order of `0`. Negative values are allowed and sort before the default.

---

## `[JsonNumberHandling]` -- Per-Property Number Handling

Override the global `NumberHandling` setting for a specific property or an entire type.

```csharp
public class Product
{
    public string Name { get; set; } = "";
    
    // This specific property accepts numbers-as-strings from the API
    [JsonNumberHandling(JsonNumberHandling.AllowReadingFromString)]
    public decimal Price { get; set; }
    
    // This property is strict (default)
    public int StockCount { get; set; }
}

// Works: {"Name":"Widget","Price":"29.99","StockCount":42}
// Price is read from string "29.99", StockCount must be a JSON number
```

Apply to an entire class to affect all numeric properties:

```csharp
[JsonNumberHandling(JsonNumberHandling.AllowReadingFromString | JsonNumberHandling.WriteAsString)]
public class FinancialRecord
{
    public decimal Amount { get; set; }      // read from string, written as string
    public decimal Tax { get; set; }         // read from string, written as string
}
```

---

## `[JsonConverter]` -- Per-Property or Per-Type Custom Converter

Apply a custom converter to a specific property or to all instances of a type.

```csharp
// ---- Per-property converter ----
public class Event
{
    public string Title { get; set; } = "";
    
    [JsonConverter(typeof(UnixEpochDateTimeConverter))]
    public DateTime Timestamp { get; set; }   // serialized as Unix epoch integer
}

// ---- Per-type converter (applies everywhere this type is used) ----
[JsonConverter(typeof(TemperatureConverter))]
public struct Temperature
{
    public double Value { get; set; }
    public string Unit { get; set; }
}
```

> [!ad-note] Converter Precedence
> When multiple converters could apply, the most specific one wins:
> 1. `[JsonConverter]` on the **property** -- highest priority
> 2. `[JsonConverter]` on the **type** (class/struct definition)
> 3. Converter added to `JsonSerializerOptions.Converters` in [[JsonSerializerOptions]]
> 4. Built-in converters -- lowest priority
>
> This lets you use a type-level converter as the default but override it for specific properties.

### Writing a Simple Custom Converter

```csharp
// Converter that reads/writes DateTime as a Unix epoch (seconds since 1970-01-01)
public class UnixEpochDateTimeConverter : JsonConverter<DateTime>
{
    private static readonly DateTime Epoch = new(1970, 1, 1, 0, 0, 0, DateTimeKind.Utc);
    
    public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, 
        JsonSerializerOptions options)
    {
        long seconds = reader.GetInt64();
        return Epoch.AddSeconds(seconds);
    }
    
    public override void Write(Utf8JsonWriter writer, DateTime value, 
        JsonSerializerOptions options)
    {
        long seconds = (long)(value.ToUniversalTime() - Epoch).TotalSeconds;
        writer.WriteNumberValue(seconds);
    }
}

// Usage:
public class LogEntry
{
    [JsonConverter(typeof(UnixEpochDateTimeConverter))]
    public DateTime Timestamp { get; set; }
    
    public string Message { get; set; } = "";
}

// Serialized: {"Timestamp":1717459200,"Message":"Server started"}
// Instead of: {"Timestamp":"2024-06-04T00:00:00Z","Message":"Server started"}
```

---

## `[JsonConstructor]` -- Specify the Deserialization Constructor

Tells `JsonSerializer` which constructor to use when creating instances during deserialization. Essential for immutable types and records with multiple constructors.

```csharp
public class Point
{
    public double X { get; }
    public double Y { get; }
    
    // This constructor is used for deserialization
    [JsonConstructor]
    public Point(double x, double y)
    {
        X = x;
        Y = y;
    }
    
    // This constructor exists for convenience but should not be used by the serializer
    public Point() : this(0, 0) { }
}
```

**How parameter matching works:**

The serializer matches constructor parameter names to JSON property names **case-insensitively**:

```csharp
public class Person
{
    public string Name { get; }
    public int Age { get; }
    
    [JsonConstructor]
    public Person(string name, int age)   // "name" matches JSON "Name" (case-insensitive)
    {
        Name = name;
        Age = age;
    }
}

// JSON: {"Name":"Long","Age":28}
// Deserialization: new Person(name: "Long", age: 28)
```

> [!ad-warning] Parameter Names Must Match Property Names
> If a constructor parameter does not match any JSON property name, the serializer passes the parameter type's default value (`null`, `0`, etc.). This is a silent failure -- no exception is thrown. Always ensure your constructor parameter names match your property names (ignoring case).

**Records work naturally:**

```csharp
// Records have a primary constructor that JsonSerializer uses automatically
public record PersonRecord(string Name, int Age);

// No [JsonConstructor] needed -- the primary constructor is selected automatically
var json = """{"Name":"Long","Age":28}""";
var person = JsonSerializer.Deserialize<PersonRecord>(json);
```

> [!ad-info] Multiple Constructors Without `[JsonConstructor]`
> If your type has multiple public constructors and none is marked with `[JsonConstructor]`, the serializer looks for a public parameterless constructor. If one exists, it uses that. If no parameterless constructor exists and there are multiple parameterized constructors, deserialization throws `NotSupportedException`. Always annotate the intended constructor.

---

## `[JsonDerivedType]` and `[JsonPolymorphic]` (.NET 7+) -- Polymorphic Serialization

These attributes enable serialization and deserialization of type hierarchies where the runtime type may differ from the declared type.

**The Problem Without These Attributes:**

```csharp
public class Shape { public string Color { get; set; } = ""; }
public class Circle : Shape { public double Radius { get; set; } }
public class Rectangle : Shape { public double Width { get; set; } public double Height { get; set; } }

Shape shape = new Circle { Color = "red", Radius = 5.0 };

// Without polymorphic support:
string json = JsonSerializer.Serialize<Shape>(shape);
// {"Color":"red"}   <-- Radius is LOST because serializer sees Shape, not Circle
```

**The Solution:**

```csharp
[JsonDerivedType(typeof(Circle), typeDiscriminator: "circle")]
[JsonDerivedType(typeof(Rectangle), typeDiscriminator: "rect")]
public class Shape
{
    public string Color { get; set; } = "";
}

public class Circle : Shape
{
    public double Radius { get; set; }
}

public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }
}

// Now serialization preserves the derived type:
Shape shape = new Circle { Color = "red", Radius = 5.0 };
string json = JsonSerializer.Serialize<Shape>(shape);
// {"$type":"circle","Color":"red","Radius":5}

// And deserialization restores the correct derived type:
Shape? restored = JsonSerializer.Deserialize<Shape>(json);
// restored is Circle { Color = "red", Radius = 5.0 }
Console.WriteLine(restored is Circle);  // true
```

**Type discriminators** can be strings or integers:

```csharp
// String discriminators (more readable):
[JsonDerivedType(typeof(Circle), typeDiscriminator: "circle")]

// Integer discriminators (more compact):
[JsonDerivedType(typeof(Circle), typeDiscriminator: 1)]
[JsonDerivedType(typeof(Rectangle), typeDiscriminator: 2)]
```

**`[JsonPolymorphic]` -- configure the discriminator property name:**

```csharp
[JsonPolymorphic(TypeDiscriminatorPropertyName = "_type")]  // default is "$type"
[JsonDerivedType(typeof(Circle), "circle")]
[JsonDerivedType(typeof(Rectangle), "rect")]
public class Shape { }

// Produces: {"_type":"circle","Color":"red","Radius":5}
// Instead of: {"$type":"circle",...}
```

> [!ad-warning] Security: `[JsonDerivedType]` Is Safe, `TypeNameHandling` Was Not
> In Newtonsoft.Json, `TypeNameHandling.All` embedded the full .NET type name in JSON, which enabled deserialization attacks (an attacker could specify any type, including `System.Diagnostics.Process`). `System.Text.Json`'s approach is **safe by design** -- only types explicitly registered with `[JsonDerivedType]` can be instantiated. The type discriminator is a user-defined string/integer, not a .NET type name.

---

## `[JsonExtensionData]` -- Capture Unmapped Properties

Collects any JSON properties that don't match a C# property into a dictionary. This is a "catch-all" for extra data.

```csharp
public class Config
{
    public string Name { get; set; } = "";
    public int Version { get; set; }
    
    [JsonExtensionData]
    public Dictionary<string, JsonElement>? AdditionalData { get; set; }
}

string json = """
{
    "Name": "MyApp",
    "Version": 2,
    "Author": "Long",
    "License": "MIT",
    "Experimental": true
}
""";

var config = JsonSerializer.Deserialize<Config>(json);
// config.Name == "MyApp"
// config.Version == 2
// config.AdditionalData contains:
//   "Author"       -> JsonElement "Long"
//   "License"      -> JsonElement "MIT"
//   "Experimental" -> JsonElement true
```

**Round-trip behavior:** When you re-serialize the object, the extension data is written back to JSON:

```csharp
string roundTripped = JsonSerializer.Serialize(config);
// {"Name":"MyApp","Version":2,"Author":"Long","License":"MIT","Experimental":true}
// All original properties are preserved, even the unmapped ones
```

> [!ad-note] Supported Dictionary Types for `[JsonExtensionData]`
> The property can be:
> - `Dictionary<string, JsonElement>` -- read-only access to extra values (most common)
> - `Dictionary<string, object?>` -- values are `JsonElement` at runtime
> - `JsonObject` (.NET 6+) -- mutable access
>
> Only **one** property per class can have `[JsonExtensionData]`. Having two throws `InvalidOperationException`.

> [!ad-info] Use Cases for `[JsonExtensionData]`
> - **Forward compatibility**: Your class can handle JSON with additional properties from a newer API version without losing them on round-trip.
> - **Flexible schemas**: When different entities have different sets of custom properties.
> - **Pass-through**: When your service receives JSON, processes some fields, and forwards the rest unchanged.

---

## `[JsonUnmappedMemberHandling]` (.NET 8+) -- Per-Type Unknown Property Handling

The per-type equivalent of `UnmappedMemberHandling` in [[JsonSerializerOptions]]:

```csharp
// This specific type rejects unknown JSON properties
[JsonUnmappedMemberHandling(JsonUnmappedMemberHandling.Disallow)]
public class StrictContract
{
    public string Name { get; set; } = "";
    public int Value { get; set; }
}

// Other types in the same project can still be lenient (default Skip behavior)
public class FlexibleContract
{
    public string Name { get; set; } = "";
    // Unknown properties are silently ignored
}
```

---

## `[JsonObjectCreationHandling]` (.NET 8+) -- Populate vs. Replace

Controls whether the serializer **replaces** a property's value entirely or **populates** an existing instance during deserialization.

```csharp
public class Dashboard
{
    // Replace (default): the serializer creates a new List and replaces the existing one
    public List<string> Widgets { get; set; } = new() { "clock", "weather" };
    
    // Populate: the serializer ADDS items to the existing list
    [JsonObjectCreationHandling(JsonObjectCreationHandling.Populate)]
    public List<string> DefaultWidgets { get; set; } = new() { "clock", "weather" };
}

string json = """{"Widgets":["news"],"DefaultWidgets":["news"]}""";
var dashboard = JsonSerializer.Deserialize<Dashboard>(json);

// dashboard.Widgets:        ["news"]                    <-- replaced entirely
// dashboard.DefaultWidgets: ["clock", "weather", "news"] <-- populated (added to existing)
```

> [!ad-note] When Populate Mode Is Useful
> - Pre-initialized collections that should keep their defaults and merge with JSON data
> - Complex objects where only some properties come from JSON and the rest should retain their initialized values
> - Working with immutable or factory-created inner objects

---

## Practical Example -- Combining Multiple Attributes

```csharp
[JsonDerivedType(typeof(Employee), "employee")]
[JsonDerivedType(typeof(Contractor), "contractor")]
public class Worker
{
    [JsonRequired]
    [JsonPropertyName("full_name")]
    public string Name { get; set; } = "";
    
    [JsonPropertyName("start_date")]
    public DateOnly StartDate { get; set; }
    
    [JsonIgnore]
    public string InternalId { get; set; } = Guid.NewGuid().ToString();
    
    [JsonConverter(typeof(JsonStringEnumConverter))]
    [JsonPropertyName("status")]
    public WorkerStatus Status { get; set; }
    
    [JsonPropertyOrder(-1)]   // appears first in output
    [JsonPropertyName("$type")]
    public string? TypeHint { get; set; }
    
    [JsonExtensionData]
    public Dictionary<string, JsonElement>? Metadata { get; set; }
}

public class Employee : Worker
{
    [JsonPropertyName("department")]
    public string Department { get; set; } = "";
    
    [JsonNumberHandling(JsonNumberHandling.AllowReadingFromString)]
    [JsonPropertyName("salary")]
    public decimal Salary { get; set; }
}

public class Contractor : Worker
{
    [JsonPropertyName("hourly_rate")]
    public decimal HourlyRate { get; set; }
    
    [JsonPropertyName("agency")]
    public string? Agency { get; set; }
}

public enum WorkerStatus { Active, OnLeave, Terminated }
```

Serialized Employee:
```json
{
  "$type": "employee",
  "full_name": "Long Pham",
  "start_date": "2023-06-15",
  "status": "Active",
  "department": "Engineering",
  "salary": 95000
}
```

Notice:
- `InternalId` is absent (`[JsonIgnore]`)
- `full_name` instead of `Name` (`[JsonPropertyName]`)
- `status` is `"Active"` not `0` (`JsonStringEnumConverter`)
- `$type` appears first (`[JsonPropertyOrder(-1)]`)
- `salary` accepts `"95000"` as string from APIs (`AllowReadingFromString`)

---

## Attribute vs. Options -- When to Use Which

| Scenario | Use Attribute | Use Options |
|---|---|---|
| One property needs a specific JSON name | `[JsonPropertyName]` | -- |
| All properties should be camelCase | -- | `PropertyNamingPolicy = CamelCase` |
| One property should never serialize | `[JsonIgnore]` | -- |
| All null values should be omitted | -- | `DefaultIgnoreCondition = WhenWritingNull` |
| One property should always include (even if null) | `[JsonIgnore(Condition = Never)]` | -- |
| All enums should be strings | -- | `Converters = { new JsonStringEnumConverter() }` |
| One enum property should be a string | `[JsonConverter(typeof(JsonStringEnumConverter))]` | -- |
| Need to handle polymorphism | `[JsonDerivedType]` on base class | -- |
| Need to control constructor for deserialization | `[JsonConstructor]` on constructor | -- |
| Need lenient parsing of third-party JSON | -- | `AllowTrailingCommas`, `ReadCommentHandling`, etc. |

The general rule: use **attributes** for per-member/per-type decisions that are intrinsic to the model, and use **options** for environmental/contextual settings that might change between serialization contexts (e.g., pretty-printing for debug but compact for production).

---

## Related Notes

- [[Object Serialization Overview]] -- What serialization is, why it matters, format comparison
- [[System.Text.Json Overview]] -- Architecture, API levels, comparison with Newtonsoft.Json
- [[JsonSerializer]] -- The serialize/deserialize API that these attributes configure
- [[JsonSerializerOptions]] -- Global configuration that attributes can override per-property
