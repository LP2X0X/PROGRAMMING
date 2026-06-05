---
tags:
  - csharp
  - serialization
  - xml
---

## XML Serialization

**`XmlSerializer`** converts C# objects to XML and back. While JSON dominates modern web APIs, XML serialization remains relevant for legacy system integration, SOAP web services, configuration files (`.csproj`, `.config`), and enterprise environments where XML schemas (XSD) are the contract format.

> [!ad-info] Namespace
> `XmlSerializer` lives in `System.Xml.Serialization`. For `DataContractSerializer`, you need `System.Runtime.Serialization`.

---

## Basic Usage

### Serialization (Object to XML)

```csharp
using System.Xml.Serialization;

var person = new Person { Name = "Long", Age = 28 };

// Create a serializer for the target type
var serializer = new XmlSerializer(typeof(Person));

// Serialize to a StringWriter
using var writer = new StringWriter();
serializer.Serialize(writer, person);
string xml = writer.ToString();

Console.WriteLine(xml);
```

Output (simplified):

```xml
<?xml version="1.0" encoding="utf-16"?>
<Person xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:xsd="http://www.w3.org/2001/XMLSchema">
  <Name>Long</Name>
  <Age>28</Age>
</Person>
```

### Deserialization (XML to Object)

```csharp
string xml = """
    <Person>
        <Name>Long</Name>
        <Age>28</Age>
    </Person>
    """;

var serializer = new XmlSerializer(typeof(Person));
using var reader = new StringReader(xml);

Person? restored = (Person?)serializer.Deserialize(reader);

Console.WriteLine($"{restored!.Name}, {restored.Age}");
// Output: Long, 28
```

### Serializing to/from Files

```csharp
// Serialize to a file
var serializer = new XmlSerializer(typeof(Person));
using (var stream = new FileStream("person.xml", FileMode.Create))
{
    serializer.Serialize(stream, person);
}

// Deserialize from a file
using (var stream = new FileStream("person.xml", FileMode.Open))
{
    Person? loaded = (Person?)serializer.Deserialize(stream);
}
```

### Serializing to/from Streams

```csharp
// To MemoryStream
using var memoryStream = new MemoryStream();
serializer.Serialize(memoryStream, person);

// Reset position to read it back
memoryStream.Position = 0;
Person? fromStream = (Person?)serializer.Deserialize(memoryStream);
```

---

## Requirements and Rules

`XmlSerializer` has specific requirements for the types it can serialize:

| Requirement | Details |
|---|---|
| **Public class** | The class must be `public` |
| **Parameterless constructor** | Must have a `public` parameterless constructor (can be implicit) |
| **Public members only** | Only `public` properties **and** `public` fields are serialized |
| **Read/write properties** | Properties must have both `get` and `set` (set can't be `init`) |
| **Supported types** | Primitives, strings, arrays, `List<T>`, `Dictionary` (with caveats), enums, nested serializable types |

> [!ad-note] Fields are included by default
> Unlike `System.Text.Json` (which only serializes public properties by default), `XmlSerializer` serializes **both** public properties and public fields. Be careful not to accidentally expose fields you didn't intend to serialize.

> [!ad-warning] No interface or abstract type support by default
> `XmlSerializer` cannot serialize properties typed as interfaces (`IList<T>`) or abstract classes unless you tell it about the concrete types via `[XmlInclude]` or constructor overloads. You'll get an `InvalidOperationException` at runtime.

---

## XML Attributes for Control

These attributes let you control how your class maps to XML elements and attributes:

| Attribute | Purpose | Applied to |
|---|---|---|
| `[XmlRoot("name")]` | Rename the root XML element | Class |
| `[XmlElement("name")]` | Rename a property's XML element | Property / Field |
| `[XmlAttribute("name")]` | Serialize as an XML attribute instead of child element | Property / Field |
| `[XmlIgnore]` | Exclude from serialization | Property / Field |
| `[XmlArray("name")]` | Rename the outer collection element | Collection property |
| `[XmlArrayItem("name")]` | Rename each item element in a collection | Collection property |
| `[XmlType("name")]` | Rename the type in schema generation | Class |
| `[XmlText]` | Serialize as the text content of the parent element | Property / Field |
| `[XmlEnum("name")]` | Map an enum member to a specific XML string | Enum member |

### Full Example with Attributes

```csharp
[XmlRoot("Employee")]  // Root element is <Employee> instead of <Person>
public class Person
{
    [XmlAttribute("id")]  // Serialize as attribute: <Employee id="1">
    public int Id { get; set; }

    [XmlElement("FullName")]  // <FullName> instead of <Name>
    public string Name { get; set; } = "";

    public int Age { get; set; }

    [XmlIgnore]  // Not included in XML output
    public string InternalCode { get; set; } = "";

    [XmlArray("PhoneNumbers")]        // Outer: <PhoneNumbers>
    [XmlArrayItem("Phone")]           // Each item: <Phone>
    public List<string> Phones { get; set; } = new();

    [XmlElement("Department")]
    public Department Dept { get; set; } = new();
}

public class Department
{
    [XmlAttribute("code")]
    public string Code { get; set; } = "";

    [XmlText]  // Serialize as text content: <Department code="ENG">Engineering</Department>
    public string Name { get; set; } = "";
}
```

Serialized XML:

```xml
<Employee id="1">
  <FullName>Long</FullName>
  <Age>28</Age>
  <PhoneNumbers>
    <Phone>555-0100</Phone>
    <Phone>555-0200</Phone>
  </PhoneNumbers>
  <Department code="ENG">Engineering</Department>
</Employee>
```

---

## Handling Enums

By default, enums serialize as their **name** (not their integer value). You can customize the XML string with `[XmlEnum]`:

```csharp
public enum Status
{
    [XmlEnum("active")]
    Active,

    [XmlEnum("inactive")]
    Inactive,

    [XmlEnum("on-hold")]
    OnHold
}

public class Account
{
    public string Owner { get; set; } = "";
    public Status Status { get; set; }
}
```

```xml
<Account>
  <Owner>Long</Owner>
  <Status>active</Status>
</Account>
```

---

## Handling Inheritance and Polymorphism

When a property is typed as a base class but holds a derived instance, you must declare the derived types:

```csharp
// Option 1: [XmlInclude] on the base class
[XmlInclude(typeof(Dog))]
[XmlInclude(typeof(Cat))]
public class Animal
{
    public string Name { get; set; } = "";
}

public class Dog : Animal
{
    public string Breed { get; set; } = "";
}

public class Cat : Animal
{
    public bool IsIndoor { get; set; }
}

public class Shelter
{
    // This can hold Dog or Cat instances
    public List<Animal> Animals { get; set; } = new();
}
```

The serialized XML uses `xsi:type` to record the actual type:

```xml
<Shelter>
  <Animals>
    <Animal xsi:type="Dog">
      <Name>Rex</Name>
      <Breed>Labrador</Breed>
    </Animal>
    <Animal xsi:type="Cat">
      <Name>Whiskers</Name>
      <IsIndoor>true</IsIndoor>
    </Animal>
  </Animals>
</Shelter>
```

---

## Controlling XML Namespaces and Declaration

### Removing the Default Namespaces

The default output includes `xmlns:xsi` and `xmlns:xsd` declarations. To remove them:

```csharp
var serializer = new XmlSerializer(typeof(Person));
var namespaces = new XmlSerializerNamespaces();
namespaces.Add("", "");  // Empty prefix and namespace

using var writer = new StringWriter();
serializer.Serialize(writer, person, namespaces);
```

### Removing the XML Declaration

To omit the `<?xml version="1.0" ...?>` line:

```csharp
var serializer = new XmlSerializer(typeof(Person));
var settings = new XmlWriterSettings
{
    OmitXmlDeclaration = true,
    Indent = true
};

using var stringWriter = new StringWriter();
using var xmlWriter = XmlWriter.Create(stringWriter, settings);
serializer.Serialize(xmlWriter, person);
```

---

## XmlSerializer Performance Considerations

> [!ad-warning] Cache your XmlSerializer instances
> When you create `new XmlSerializer(typeof(T))`, the runtime dynamically generates and compiles a temporary assembly to handle serialization for that type. This is **expensive** on first call. Always cache and reuse `XmlSerializer` instances:
> ```csharp
> // DO -- cache in a static field
> private static readonly XmlSerializer s_personSerializer = new(typeof(Person));
> 
> // DON'T -- creates a new assembly every call (memory leak potential)
> var serializer = new XmlSerializer(typeof(Person)); // in a loop or method
> ```
> The simple constructors (`new XmlSerializer(typeof(T))`) cache internally, so repeated calls with the same type are safe. But the overloads that take extra parameters (`extraTypes`, `XmlAttributeOverrides`, etc.) do **not** cache and **will** leak assemblies.

---

## DataContractSerializer (Brief Overview)

`DataContractSerializer` is an alternative XML serializer that originated in WCF. Key differences from `XmlSerializer`:

| | `XmlSerializer` | `DataContractSerializer` |
|---|---|---|
| **Namespace** | `System.Xml.Serialization` | `System.Runtime.Serialization` |
| **Opt-in model** | Serializes all public members | Only members marked `[DataMember]` |
| **Private members** | Not supported | Supported (with `[DataMember]`) |
| **Parameterless ctor** | Required | Not required |
| **Circular references** | Not supported | Supported (`IsReference = true`) |
| **Ordering** | XML element order matches class | Explicit order via `[DataMember(Order = n)]` |
| **Use case** | General XML, configuration | WCF services, precise control |

```csharp
using System.Runtime.Serialization;

[DataContract(Name = "Employee")]
public class Person
{
    [DataMember(Order = 1)]
    public string Name { get; set; } = "";

    [DataMember(Order = 2)]
    public int Age { get; set; }

    // Not marked [DataMember] -- will NOT be serialized
    public string InternalCode { get; set; } = "";

    [DataMember(Order = 3)]
    private string _secret = "hidden";  // Private members CAN be serialized
}
```

---

## XmlSerializer vs System.Text.Json

| | `XmlSerializer` | `System.Text.Json` |
|---|---|---|
| **Output format** | XML | JSON |
| **Performance** | Slower (assembly generation, XML parsing) | Faster (UTF-8, source generators) |
| **Parameterless ctor** | Required | Required by default (configurable) |
| **Public fields** | Included by default | Excluded by default |
| **Private members** | Not supported | Not supported by default |
| **Streaming** | Via `XmlReader`/`XmlWriter` | Via `Utf8JsonReader`/`Utf8JsonWriter` |
| **Schema support** | XSD | JSON Schema (external tools) |
| **Use when** | XML APIs, SOAP, legacy, enterprise | Web APIs, REST, modern apps, config |

---

## See Also

- [[Custom JsonConverters]] -- writing custom serialization logic for JSON
- [[JsonDocument and JsonNode]] -- working with JSON without a strongly-typed class
- [[BinaryFormatter and Modern Alternatives]] -- binary serialization options and why `BinaryFormatter` is gone
