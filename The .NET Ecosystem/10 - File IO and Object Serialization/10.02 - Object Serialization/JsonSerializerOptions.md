---
tags:
 - csharp
 - serialization
 - json
---

## JsonSerializerOptions -- Configuring How JsonSerializer Behaves

`JsonSerializerOptions` is the configuration object you pass to `JsonSerializer.Serialize()` and `JsonSerializer.Deserialize()` to control every aspect of the serialization process -- naming conventions, null handling, indentation, circular references, custom converters, and much more.

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;
```

---

## Creating and Reusing Options

```csharp
var options = new JsonSerializerOptions
{
    WriteIndented = true,
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull
};

// Use the same instance everywhere
string json = JsonSerializer.Serialize(person, options);
Person? p = JsonSerializer.Deserialize<Person>(json, options);
```

> [!ad-important] Create Options ONCE and Reuse
> The `JsonSerializerOptions` constructor is **expensive**. On first use with a given type, it builds internal metadata caches (reflection data, converter lookups, property mappings) and stores them on the options instance. If you create a new `JsonSerializerOptions` on every call, you:
> 1. Pay the full cache-building cost every time
> 2. Generate significant garbage for the GC to collect
> 3. Miss out on the warm-cache fast path
>
> **Always** store your options in a `static readonly` field or a singleton:
> ```csharp
> public static class JsonConfig
> {
>     public static readonly JsonSerializerOptions Default = new()
>     {
>         WriteIndented = true,
>         PropertyNamingPolicy = JsonNamingPolicy.CamelCase
>     };
> }
> 
> // Usage
> string json = JsonSerializer.Serialize(person, JsonConfig.Default);
> ```

> [!ad-warning] Options Are Frozen After First Use
> Once you pass a `JsonSerializerOptions` instance to any `JsonSerializer` method, it becomes **frozen** (immutable). Any attempt to modify a property after that point throws `InvalidOperationException`. This is by design -- it protects the internal caches from corruption. If you need different options, create a new instance:
> ```csharp
> // Copy constructor -- creates a mutable clone of existing options
> var newOptions = new JsonSerializerOptions(existingOptions);
> newOptions.WriteIndented = false;  // OK -- this is a new, unfrozen instance
> ```

---

## Web Defaults Shortcut (.NET 6+)

ASP.NET Core uses a specific set of options for web APIs. You can get the same defaults without configuring each property manually:

```csharp
var webOptions = new JsonSerializerOptions(JsonSerializerDefaults.Web);
// Equivalent to:
// {
//     PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
//     PropertyNameCaseInsensitive = true,
//     NumberHandling = JsonNumberHandling.AllowReadingFromString
// }
```

This is useful when you want your non-ASP.NET code (console apps, background services) to behave the same as your web API serialization.

| Preset | `JsonSerializerDefaults.General` (default) | `JsonSerializerDefaults.Web` |
|---|---|---|
| Naming policy | `null` (PascalCase) | `CamelCase` |
| Case-insensitive reads | `false` | `true` |
| Number from string | `Strict` | `AllowReadingFromString` |

---

## Full Property Reference

### Formatting

| Property | Type | Default | What It Does |
|---|---|---|---|
| `WriteIndented` | `bool` | `false` | Pretty-print JSON with indentation and line breaks |
| `IndentCharacter` | `char` | `' '` (space) | Character used for indentation (.NET 9+) |
| `IndentSize` | `int` | `2` | Number of indent characters per level (.NET 9+) |
| `NewLine` | `string` | `"\n"` | Line ending used in indented output (.NET 9+) |
| `Encoder` | `JavaScriptEncoder?` | default (strict) | Controls which characters are escaped in strings |

```csharp
// Pretty-printed with 4-space indentation
var options = new JsonSerializerOptions
{
    WriteIndented = true
    // In .NET 9+: IndentSize = 4
};
```

**Encoder -- Relaxing HTML-Safe Escaping:**

By default, `System.Text.Json` aggressively escapes characters that could be dangerous in HTML contexts (like `<`, `>`, `&`, `'`, and non-ASCII characters). This produces safe but ugly output:

```csharp
var data = new { Message = "Hello <World> & 'Friends'" };

// Default encoder: HTML-safe but verbose
JsonSerializer.Serialize(data);
// {"Message":"Hello <World> & 'Friends'"}

// Relaxed encoder: more readable, still valid JSON
var relaxed = new JsonSerializerOptions
{
    Encoder = System.Text.Encodings.Web.JavaScriptEncoder.UnsafeRelaxedJsonEscaping
};
JsonSerializer.Serialize(data, relaxed);
// {"Message":"Hello <World> & 'Friends'"}
```

> [!ad-warning] UnsafeRelaxedJsonEscaping is "Unsafe" for a Reason
> The name includes "Unsafe" because the output may contain characters that are dangerous if embedded directly in HTML or JavaScript. Only use it when you control the consumer (e.g., writing to a file or an API response with proper `Content-Type` headers). Never use it when embedding JSON inside an HTML `<script>` tag.

---

### Naming

| Property | Type | Default | What It Does |
|---|---|---|---|
| `PropertyNamingPolicy` | `JsonNamingPolicy?` | `null` (PascalCase) | Transform property names during serialization |
| `PropertyNameCaseInsensitive` | `bool` | `false` | Case-insensitive matching during deserialization |
| `DictionaryKeyPolicy` | `JsonNamingPolicy?` | `null` | Transform dictionary keys (separate from property names) |

**Built-in Naming Policies:**

| Policy | C# Property `FirstName` Becomes | Available Since |
|---|---|---|
| `null` (default) | `"FirstName"` | .NET Core 3.0 |
| `JsonNamingPolicy.CamelCase` | `"firstName"` | .NET Core 3.0 |
| `JsonNamingPolicy.SnakeCaseLower` | `"first_name"` | .NET 8 |
| `JsonNamingPolicy.SnakeCaseUpper` | `"FIRST_NAME"` | .NET 8 |
| `JsonNamingPolicy.KebabCaseLower` | `"first-name"` | .NET 8 |
| `JsonNamingPolicy.KebabCaseUpper` | `"FIRST-NAME"` | .NET 8 |

```csharp
// CamelCase -- the most common for web APIs
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
};

var person = new Person { FirstName = "Long", LastName = "Pham" };
string json = JsonSerializer.Serialize(person, options);
// {"firstName":"Long","lastName":"Pham"}
```

> [!ad-note] `[JsonPropertyName]` Overrides the Naming Policy
> If a property has `[JsonPropertyName("custom_name")]`, that name is used regardless of the naming policy. See [[Serialization Attributes]].

> [!ad-info] Custom Naming Policy
> You can create your own by subclassing `JsonNamingPolicy`:
> ```csharp
> public class UpperCaseNamingPolicy : JsonNamingPolicy
> {
>     public override string ConvertName(string name) => name.ToUpperInvariant();
> }
> 
> var options = new JsonSerializerOptions
> {
>     PropertyNamingPolicy = new UpperCaseNamingPolicy()
> };
> // Property "Name" becomes "NAME"
> ```

---

### Null and Default Value Handling

| Property | Type | Default | What It Does |
|---|---|---|---|
| `DefaultIgnoreCondition` | `JsonIgnoreCondition` | `Never` | Controls when properties are omitted from serialized output |

**JsonIgnoreCondition Values:**

| Value | Behavior |
|---|---|
| `Never` | Always include the property, even if `null` or default |
| `WhenWritingNull` | Omit properties that are `null` |
| `WhenWritingDefault` | Omit properties that are `null` OR equal to the type's default (`0` for `int`, `false` for `bool`, etc.) |
| `Always` | Never include the property (same as `[JsonIgnore]` on every property -- not useful globally) |

```csharp
public class Person
{
    public string? Name { get; set; }
    public int Age { get; set; }
    public bool IsActive { get; set; }
}

var person = new Person { Name = null, Age = 0, IsActive = false };

// Default (Never): include everything
// {"Name":null,"Age":0,"IsActive":false}

// WhenWritingNull: omit nulls only
// {"Age":0,"IsActive":false}

// WhenWritingDefault: omit nulls AND defaults
// {}   <-- all values are their type's default
```

> [!ad-warning] `WhenWritingDefault` Can Cause Silent Data Loss
> Be careful with `WhenWritingDefault`. If `Age = 0` is a meaningful value (not "missing"), `WhenWritingDefault` will omit it from the JSON. The deserializing side will also get `0` (from the default constructor), so it works out -- but only by coincidence. If the receiving side has a different default, you have a bug. Prefer `WhenWritingNull` unless you are certain defaults are always "absent."

---

### Number Handling

| Property | Type | Default | What It Does |
|---|---|---|---|
| `NumberHandling` | `JsonNumberHandling` | `Strict` | How to read/write numbers |

**JsonNumberHandling Values (flags -- can be combined):**

| Value | Behavior |
|---|---|
| `Strict` | Numbers must be JSON numbers, not strings |
| `AllowReadingFromString` | `"42"` can be read as `int 42` |
| `WriteAsString` | Write numbers as `"42"` instead of `42` |
| `AllowNamedFloatingPointLiterals` | Allow `"NaN"`, `"Infinity"`, `"-Infinity"` |

```csharp
var options = new JsonSerializerOptions
{
    NumberHandling = JsonNumberHandling.AllowReadingFromString
};

// This now works:
string json = """{"Count":"42"}""";
var obj = JsonSerializer.Deserialize<MyClass>(json, options);
// obj.Count == 42 (int)
```

> [!ad-note] Why AllowReadingFromString Exists
> Many real-world APIs (especially JavaScript-based ones) send numbers as strings to avoid floating-point precision issues with large numbers. `AllowReadingFromString` is essential for interoperability with these APIs.

---

### Reference Handling (Circular References)

| Property | Type | Default | What It Does |
|---|---|---|---|
| `ReferenceHandler` | `ReferenceHandler?` | `null` | How to handle circular references and duplicate object references |

Without configuration, circular references throw a `JsonException`. You have two options:

**Option 1: `IgnoreCycles` (.NET 6+) -- replace cycles with `null`**

```csharp
var options = new JsonSerializerOptions
{
    ReferenceHandler = ReferenceHandler.IgnoreCycles
};
```

When the serializer encounters an object it has already visited in the current graph traversal, it writes `null` instead of entering an infinite loop. This is the simplest fix but you lose data -- the back-reference becomes `null`.

**Option 2: `Preserve` -- use `$id` / `$ref` metadata**

```csharp
var options = new JsonSerializerOptions
{
    ReferenceHandler = ReferenceHandler.Preserve
};

// Produces JSON like:
// {
//   "$id": "1",
//   "Name": "Long",
//   "Manager": {
//     "$id": "2",
//     "Name": "Alice",
//     "Reports": [
//       { "$ref": "1" }     <-- refers back to Long without repeating
//     ]
//   }
// }
```

`Preserve` produces lossless output but the JSON contains `$id` and `$ref` metadata that non-.NET consumers may not understand.

| Strategy | Lossy? | Standard JSON? | Best For |
|---|---|---|---|
| `IgnoreCycles` | Yes (cycles become `null`) | Yes | Simple cases, web APIs where consumers don't expect `$ref` |
| `Preserve` | No | No (`$id`/`$ref` are non-standard) | .NET-to-.NET communication where roundtrip fidelity matters |
| Break with `[JsonIgnore]` | Depends | Yes | Cleanest solution -- eliminate the cycle at the model level |

> [!ad-info] Preferred Approach: Eliminate the Cycle
> The cleanest solution is to break the cycle in your data model by placing `[JsonIgnore]` on the back-reference property. This avoids any configuration and produces clean, standard JSON. Use `IgnoreCycles` or `Preserve` only when you cannot change the model. See [[Serialization Attributes]].

---

### Deserialization Strictness

| Property | Type | Default | What It Does |
|---|---|---|---|
| `AllowTrailingCommas` | `bool` | `false` | Allow trailing commas in JSON arrays and objects |
| `ReadCommentHandling` | `JsonCommentHandling` | `Disallow` | How to handle `//` and `/* */` comments in JSON |
| `MaxDepth` | `int` | `64` | Maximum nesting depth allowed |
| `UnmappedMemberHandling` | `JsonUnmappedMemberHandling` | `Skip` | What to do with JSON properties that don't match a C# property (.NET 8+) |
| `PreferredObjectCreationHandling` | `JsonObjectCreationHandling` | `Replace` | Replace or populate existing objects during deserialization (.NET 8+) |

```csharp
// Lenient parsing -- useful when consuming messy JSON from third parties
var lenient = new JsonSerializerOptions
{
    AllowTrailingCommas = true,                              // [1, 2, 3,] is OK
    ReadCommentHandling = JsonCommentHandling.Skip,          // // comments are skipped
    NumberHandling = JsonNumberHandling.AllowReadingFromString,
    PropertyNameCaseInsensitive = true
};

// This JSON would fail with default options but succeeds with lenient:
string messyJson = """
{
    // This is a comment
    "name": "Long",
    "Age": "28",   // age as string, trailing comma
}
""";
var person = JsonSerializer.Deserialize<Person>(messyJson, lenient);
```

**UnmappedMemberHandling (.NET 8+):**

```csharp
// Strict mode -- throw if JSON has properties not in the C# class
var strict = new JsonSerializerOptions
{
    UnmappedMemberHandling = JsonUnmappedMemberHandling.Disallow
};

// JSON has "email" but Person has no Email property --> throws JsonException
string json = """{"Name":"Long","email":"long@example.com"}""";
var person = JsonSerializer.Deserialize<Person>(json, strict);  // THROWS
```

> [!ad-note] When to Disallow Unmapped Members
> Use `Disallow` when you want to catch typos or version mismatches between your API contract and the JSON payload. Use `Skip` (default) when you expect the JSON to contain extra fields you don't care about (common when consuming third-party APIs that may add fields over time).

---

### Fields and Misc

| Property | Type | Default | What It Does |
|---|---|---|---|
| `IncludeFields` | `bool` | `false` | Serialize/deserialize public fields (not just properties) |
| `DefaultBufferSize` | `int` | `16384` | Buffer size for stream-based operations |
| `IgnoreReadOnlyProperties` | `bool` | `false` | Skip read-only properties during serialization |
| `IgnoreReadOnlyFields` | `bool` | `false` | Skip read-only fields during serialization |
| `TypeInfoResolver` | `IJsonTypeInfoResolver?` | `null` | Custom metadata resolver (advanced / source generation) |

---

### Converters

| Property | Type | Default | What It Does |
|---|---|---|---|
| `Converters` | `IList<JsonConverter>` | empty | Custom converters applied globally |

```csharp
var options = new JsonSerializerOptions
{
    Converters =
    {
        new JsonStringEnumConverter(),                    // all enums as strings
        new MyCustomDateConverter(),                       // custom date format
        new JsonStringEnumConverter(JsonNamingPolicy.CamelCase)  // camelCase enum values
    }
};
```

Converter precedence (from highest to lowest):
1. `[JsonConverter]` attribute on the **property** (see [[Serialization Attributes]])
2. `[JsonConverter]` attribute on the **type**
3. Converter added to `JsonSerializerOptions.Converters` collection
4. Built-in converters

---

## Common Configuration Patterns

### Web API Configuration

```csharp
public static readonly JsonSerializerOptions WebApi = new()
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
    Converters = { new JsonStringEnumConverter(JsonNamingPolicy.CamelCase) },
    PropertyNameCaseInsensitive = true,
    NumberHandling = JsonNumberHandling.AllowReadingFromString
};
```

### File Storage / Debug Configuration

```csharp
public static readonly JsonSerializerOptions FilePretty = new()
{
    WriteIndented = true,
    Encoder = System.Text.Encodings.Web.JavaScriptEncoder.UnsafeRelaxedJsonEscaping,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull
};
```

### Strict Contract Enforcement (.NET 8+)

```csharp
public static readonly JsonSerializerOptions Strict = new()
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    UnmappedMemberHandling = JsonUnmappedMemberHandling.Disallow,
    NumberHandling = JsonNumberHandling.Strict,
    AllowTrailingCommas = false,
    ReadCommentHandling = JsonCommentHandling.Disallow
};
```

### Lenient Third-Party API Consumer

```csharp
public static readonly JsonSerializerOptions Lenient = new()
{
    PropertyNameCaseInsensitive = true,
    NumberHandling = JsonNumberHandling.AllowReadingFromString,
    AllowTrailingCommas = true,
    ReadCommentHandling = JsonCommentHandling.Skip,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull,
    UnmappedMemberHandling = JsonUnmappedMemberHandling.Skip
};
```

---

## Copying Options

When you need a variation of existing options:

```csharp
// Copy constructor creates a mutable clone
var baseOptions = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase,
    DefaultIgnoreCondition = JsonIgnoreCondition.WhenWritingNull
};

// Create a variant with indentation
var indentedOptions = new JsonSerializerOptions(baseOptions)
{
    WriteIndented = true   // add indentation to the copy
};
```

> [!ad-note] The Copy Constructor Copies Everything
> The copy constructor clones all settings, including the `Converters` list. Changes to the copy's `Converters` do not affect the original, and vice versa. The copy starts unfrozen even if the original was frozen.

---

## Related Notes

- [[Object Serialization Overview]] -- What serialization is, why it matters, format comparison
- [[System.Text.Json Overview]] -- Architecture, API levels, comparison with Newtonsoft.Json
- [[JsonSerializer]] -- The serialize/deserialize API that consumes these options
- [[Serialization Attributes]] -- Per-property control that complements global options
