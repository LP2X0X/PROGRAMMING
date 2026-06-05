---
tags:
  - csharp
  - serialization
  - json
---

## Custom JsonConverters

A **`JsonConverter<T>`** is a class you write to override how `System.Text.Json` serializes and deserializes a specific type `T`. When the default behavior doesn't match what you need, a custom converter gives you full control over reading tokens from JSON and writing tokens to JSON.

> [!ad-info] Core idea
> A custom converter intercepts the serialization pipeline for one type (or a family of types) and replaces the default logic with your own `Read()` and `Write()` methods that operate directly on `Utf8JsonReader` and `Utf8JsonWriter`.

---

## When You Need a Custom Converter

Not every type needs one. You need a custom converter when:

1. **A type doesn't serialize correctly by default** -- `DateOnly`, `TimeOnly` (pre-.NET 7), `TimeSpan`, `IPAddress`, or any type without a built-in converter
2. **You need a non-standard format** -- Unix timestamps instead of ISO 8601, custom date strings like `"dd/MM/yyyy"`, or compact representations
3. **Polymorphic deserialization (pre-.NET 7)** -- before `[JsonDerivedType]` existed, converters were the only way to handle base-type references that could be any derived type
4. **Third-party types you can't modify** -- you can't add `[JsonPropertyName]` or other attributes to types from NuGet packages or framework libraries
5. **Custom enum serialization** -- mapping enum values to specific string names that don't match the member names
6. **Data cleaning on deserialization** -- trimming whitespace, coercing `"null"` strings to `null`, normalizing formats on the way in

---

## Creating a Custom Converter

Inherit from `JsonConverter<T>` and override two methods: `Read` (deserialization) and `Write` (serialization).

### DateOnly Converter Example

```csharp
using System.Text.Json;
using System.Text.Json.Serialization;

public class DateOnlyConverter : JsonConverter<DateOnly>
{
    private const string Format = "yyyy-MM-dd";

    // Called during deserialization -- read the JSON token and return a DateOnly
    public override DateOnly Read(
        ref Utf8JsonReader reader,
        Type typeToConvert,
        JsonSerializerOptions options)
    {
        // reader.GetString() reads the current token as a string
        return DateOnly.ParseExact(reader.GetString()!, Format);
    }

    // Called during serialization -- write the DateOnly as a JSON token
    public override void Write(
        Utf8JsonWriter writer,
        DateOnly value,
        JsonSerializerOptions options)
    {
        writer.WriteStringValue(value.ToString(Format));
    }
}
```

### Unix Timestamp to DateTime Converter

```csharp
public class UnixTimestampConverter : JsonConverter<DateTime>
{
    public override DateTime Read(
        ref Utf8JsonReader reader,
        Type typeToConvert,
        JsonSerializerOptions options)
    {
        // Unix timestamp is seconds since 1970-01-01 UTC
        long seconds = reader.GetInt64();
        return DateTimeOffset.FromUnixTimeSeconds(seconds).UtcDateTime;
    }

    public override void Write(
        Utf8JsonWriter writer,
        DateTime value,
        JsonSerializerOptions options)
    {
        long seconds = new DateTimeOffset(value).ToUnixTimeSeconds();
        writer.WriteNumberValue(seconds);
    }
}
```

### Trim Whitespace on Deserialization

```csharp
public class TrimStringConverter : JsonConverter<string>
{
    public override string? Read(
        ref Utf8JsonReader reader,
        Type typeToConvert,
        JsonSerializerOptions options)
    {
        // Handle null JSON token
        if (reader.TokenType == JsonTokenType.Null)
            return null;

        return reader.GetString()?.Trim();
    }

    public override void Write(
        Utf8JsonWriter writer,
        string value,
        JsonSerializerOptions options)
    {
        // Write as-is during serialization
        writer.WriteStringValue(value);
    }
}
```

### Handling `"null"` String as Actual Null

```csharp
public class NullStringConverter : JsonConverter<string>
{
    public override string? Read(
        ref Utf8JsonReader reader,
        Type typeToConvert,
        JsonSerializerOptions options)
    {
        if (reader.TokenType == JsonTokenType.Null)
            return null;

        string? value = reader.GetString();

        // Treat the literal string "null" as actual null
        return string.Equals(value, "null", StringComparison.OrdinalIgnoreCase)
            ? null
            : value;
    }

    public override void Write(
        Utf8JsonWriter writer,
        string value,
        JsonSerializerOptions options)
    {
        writer.WriteStringValue(value);
    }
}
```

### Enum with Custom String Values

```csharp
// Suppose you have this enum:
public enum Priority { Low, Medium, High, Critical }

// But the JSON uses: "low_priority", "medium_priority", etc.
public class PriorityConverter : JsonConverter<Priority>
{
    private static readonly Dictionary<string, Priority> StringToEnum = new()
    {
        ["low_priority"]      = Priority.Low,
        ["medium_priority"]   = Priority.Medium,
        ["high_priority"]     = Priority.High,
        ["critical_priority"] = Priority.Critical,
    };

    // Reverse lookup
    private static readonly Dictionary<Priority, string> EnumToString =
        StringToEnum.ToDictionary(kv => kv.Value, kv => kv.Key);

    public override Priority Read(
        ref Utf8JsonReader reader,
        Type typeToConvert,
        JsonSerializerOptions options)
    {
        string? raw = reader.GetString();
        if (raw is not null && StringToEnum.TryGetValue(raw, out Priority result))
            return result;

        throw new JsonException($"Unknown priority value: {raw}");
    }

    public override void Write(
        Utf8JsonWriter writer,
        Priority value,
        JsonSerializerOptions options)
    {
        writer.WriteStringValue(EnumToString[value]);
    }
}
```

> [!ad-warning] Null handling in converters
> Custom converters **must** handle `null` correctly. Always check `reader.TokenType == JsonTokenType.Null` at the start of your `Read` method. If you don't, calling `reader.GetString()` on a null token will return `null`, but calling `reader.GetInt32()` will throw. For value-type converters (e.g., `JsonConverter<int>`), the serializer handles `null` before calling your converter, but for reference types you are responsible.

---

## Registering a Converter

There are three ways to register a custom converter, listed from **highest to lowest priority**:

### 1. On the Property (Attribute)

```csharp
public class Event
{
    public string Name { get; set; }

    [JsonConverter(typeof(DateOnlyConverter))]  // Only this property uses it
    public DateOnly Date { get; set; }
}
```

### 2. On the Type (Attribute)

```csharp
[JsonConverter(typeof(PriorityConverter))]  // All Priority values use this converter
public enum Priority { Low, Medium, High, Critical }
```

### 3. On the Options (Runtime)

```csharp
var options = new JsonSerializerOptions();
options.Converters.Add(new DateOnlyConverter());     // All DateOnly values use it
options.Converters.Add(new TrimStringConverter());   // All string values get trimmed

string json = JsonSerializer.Serialize(myObject, options);
```

### Registration Priority Order

When multiple converters could apply, the serializer picks the **most specific** one:

| Priority | Registration Method | Scope |
|:--------:|---------------------|-------|
| 1 (highest) | `[JsonConverter]` on the **property** | That property only |
| 2 | `[JsonConverter]` on the **type** | All instances of that type |
| 3 (lowest) | `options.Converters.Add(...)` | All instances of that type within those options |

> [!ad-note] Tip
> If you register a converter on the options and also put `[JsonConverter]` on a property, the attribute wins. This lets you set a global default and override it per-property.

---

## JsonConverter\<T\> vs JsonConverterFactory

`System.Text.Json` provides two base classes for custom conversion logic:

| | `JsonConverter<T>` | `JsonConverterFactory` |
|---|---|---|
| **Handles** | One specific type `T` | A **family** of types |
| **Override** | `Read()` and `Write()` | `CanConvert()` and `CreateConverter()` |
| **Use when** | You know the exact type at compile time | You need to handle generic types, all enums, all nullable wrappers, etc. |

### When to Use a Factory

You need a factory when the converter needs to work with **open generic types** or a **category of types**. For example:

- A converter for `List<T>` where `T` can be anything
- A converter for all enum types that serializes them as lowercase strings
- A converter for `Nullable<T>` wrapping

### Factory Example: Lowercase Enum Strings for All Enums

```csharp
public class LowercaseEnumConverterFactory : JsonConverterFactory
{
    // Called to check if this factory handles the requested type
    public override bool CanConvert(Type typeToConvert)
    {
        return typeToConvert.IsEnum;
    }

    // Called once per type -- creates a specific converter for that enum type
    public override JsonConverter CreateConverter(
        Type typeToConvert,
        JsonSerializerOptions options)
    {
        // Create a LowercaseEnumConverter<TEnum> via reflection
        Type converterType = typeof(LowercaseEnumConverter<>)
            .MakeGenericType(typeToConvert);

        return (JsonConverter)Activator.CreateInstance(converterType)!;
    }

    // Inner generic converter that does the actual work
    private class LowercaseEnumConverter<TEnum> : JsonConverter<TEnum>
        where TEnum : struct, Enum
    {
        public override TEnum Read(
            ref Utf8JsonReader reader,
            Type typeToConvert,
            JsonSerializerOptions options)
        {
            string? value = reader.GetString();
            if (Enum.TryParse<TEnum>(value, ignoreCase: true, out TEnum result))
                return result;

            throw new JsonException($"Unable to parse '{value}' as {typeof(TEnum).Name}");
        }

        public override void Write(
            Utf8JsonWriter writer,
            TEnum value,
            JsonSerializerOptions options)
        {
            writer.WriteStringValue(value.ToString().ToLowerInvariant());
        }
    }
}
```

Register the factory just like a regular converter:

```csharp
var options = new JsonSerializerOptions();
options.Converters.Add(new LowercaseEnumConverterFactory());

// Now ALL enums serialize as lowercase strings
```

---

## Working with Utf8JsonReader and Utf8JsonWriter

Inside your converter methods, you interact directly with these low-level types. Understanding their API is essential.

### Utf8JsonReader (Used in `Read`)

| Member | Purpose |
|--------|---------|
| `reader.TokenType` | Current token: `StartObject`, `EndObject`, `StartArray`, `EndArray`, `PropertyName`, `String`, `Number`, `True`, `False`, `Null` |
| `reader.GetString()` | Get current token as `string` |
| `reader.GetInt32()` | Get current token as `int` |
| `reader.GetInt64()` | Get current token as `long` |
| `reader.GetDouble()` | Get current token as `double` |
| `reader.GetBoolean()` | Get current token as `bool` |
| `reader.GetDecimal()` | Get current token as `decimal` |
| `reader.Read()` | Advance to the next token (returns `true` if successful) |
| `reader.TryGetInt32(out int v)` | Try to parse as `int` without throwing |

### Utf8JsonWriter (Used in `Write`)

| Member | Purpose |
|--------|---------|
| `writer.WriteStringValue(string)` | Write a JSON string value |
| `writer.WriteNumberValue(int/long/double)` | Write a JSON number |
| `writer.WriteBooleanValue(bool)` | Write `true` or `false` |
| `writer.WriteNullValue()` | Write `null` |
| `writer.WriteStartObject()` | Write `{` |
| `writer.WriteEndObject()` | Write `}` |
| `writer.WriteStartArray()` | Write `[` |
| `writer.WriteEndArray()` | Write `]` |
| `writer.WritePropertyName(string)` | Write a property name (before its value) |

### Converter for a Complex Object

When your converter handles an object type (not just a primitive), you need to read/write the full structure:

```csharp
public class PointConverter : JsonConverter<Point>
{
    public override Point Read(
        ref Utf8JsonReader reader,
        Type typeToConvert,
        JsonSerializerOptions options)
    {
        // Expect: { "X": 10, "Y": 20 }
        if (reader.TokenType != JsonTokenType.StartObject)
            throw new JsonException("Expected StartObject");

        int x = 0, y = 0;

        while (reader.Read())
        {
            if (reader.TokenType == JsonTokenType.EndObject)
                return new Point(x, y);

            if (reader.TokenType != JsonTokenType.PropertyName)
                throw new JsonException("Expected PropertyName");

            string property = reader.GetString()!;
            reader.Read(); // advance to the value

            switch (property)
            {
                case "X": x = reader.GetInt32(); break;
                case "Y": y = reader.GetInt32(); break;
                default: reader.Skip(); break;  // skip unknown properties
            }
        }

        throw new JsonException("Unexpected end of JSON");
    }

    public override void Write(
        Utf8JsonWriter writer,
        Point value,
        JsonSerializerOptions options)
    {
        writer.WriteStartObject();
        writer.WriteNumber("X", value.X);  // WriteNumber is shorthand for
        writer.WriteNumber("Y", value.Y);  // WritePropertyName + WriteNumberValue
        writer.WriteEndObject();
    }
}
```

> [!ad-important] Reader positioning
> When your `Read` method is called, the reader is **already positioned on the first token** of your type (e.g., `StartObject` or `String`). You do **not** need to call `reader.Read()` before inspecting `reader.TokenType`. When you return from `Read`, the reader must be positioned on the **last token** of your type (e.g., `EndObject`). The serializer calls `reader.Read()` after your method returns to advance past it.

---

## See Also

- [[JsonDocument and JsonNode]] -- working with JSON without a strongly-typed class
- [[XML Serialization]] -- XML-based serialization with `XmlSerializer`
- [[BinaryFormatter and Modern Alternatives]] -- binary serialization options
