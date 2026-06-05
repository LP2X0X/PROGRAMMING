---
tags:
  - csharp
  - serialization
  - data-formats
---

## JSON and XML Structure

Understanding the structure of JSON and XML is essential before working with serialization, because serialization is the process of converting objects **into** these formats. If you don't know the rules of the target format, debugging serialization output becomes guesswork.

---

## JSON Structure

**JSON** (JavaScript Object Notation) is a lightweight, text-based data interchange format. Despite the name, it is language-independent and used everywhere.

### Data Types

JSON supports exactly **six** data types:

| Type | Example | Notes |
|---|---|---|
| String | `"hello"` | Must use **double quotes** only |
| Number | `42`, `3.14`, `-1`, `2e10` | No distinction between int/float. No `NaN` or `Infinity` |
| Boolean | `true`, `false` | Lowercase only |
| Null | `null` | Lowercase only |
| Object | `{ "key": "value" }` | Unordered set of key-value pairs |
| Array | `[1, 2, 3]` | Ordered list of values |

```ad-warning
title: No Date Type
JSON has **no native date type**. Dates are typically serialized as ISO 8601 strings (`"2026-06-04T10:30:00Z"`). This is a common source of bugs when deserializing — you must configure your serializer to handle date formats correctly. See [[JsonSerializerOptions]] for how `System.Text.Json` handles this.
```

### Syntax Rules

- Keys **must** be double-quoted strings — `{ name: "John" }` is **invalid**
- Values are separated by **commas**
- **No trailing commas** — `{ "a": 1, }` is invalid JSON (even though JavaScript allows it)
- **No comments** — JSON does not support `//` or `/* */` comments
- Whitespace (spaces, tabs, newlines) is insignificant and used only for readability
- Strings use backslash escaping: `\"`, `\\`, `\n`, `\t`, `\uXXXX`

### Object

An unordered collection of key-value pairs enclosed in curly braces:

```json
{
    "firstName": "Long",
    "age": 28,
    "isActive": true
}
```

### Array

An ordered list of values enclosed in square brackets. Values do **not** need to be the same type (though in practice they usually are):

```json
["C#", "SQL", "TypeScript"]
```

```json
[42, "mixed", true, null]
```

### Nested Example

A realistic JSON document showing nesting of objects and arrays:

```json
{
    "orderId": 1001,
    "customer": {
        "name": "Long Pham",
        "email": "long@example.com",
        "addresses": [
            {
                "type": "billing",
                "city": "Ho Chi Minh City",
                "country": "Vietnam"
            },
            {
                "type": "shipping",
                "city": "Da Nang",
                "country": "Vietnam"
            }
        ]
    },
    "items": [
        { "product": "Keyboard", "qty": 1, "price": 79.99 },
        { "product": "Mouse", "qty": 2, "price": 29.99 }
    ],
    "totalAmount": 139.97,
    "isPaid": false,
    "notes": null
}
```

### Common Uses

- **Web APIs** — the dominant format for REST APIs (JSON over HTTP)
- **Configuration files** — `appsettings.json`, `package.json`, `tsconfig.json`
- **Data storage** — NoSQL databases like MongoDB store JSON-like documents (BSON)
- **Inter-process communication** — lightweight message format between services

---

## XML Structure

**XML** (eXtensible Markup Language) is a markup language for encoding documents in a format that is both human-readable and machine-readable. It is more verbose than JSON but supports features JSON does not (attributes, comments, schemas, namespaces).

### XML Prolog

The optional (but recommended) declaration at the top of an XML document:

```xml
<?xml version="1.0" encoding="UTF-8"?>
```

- `version` — always `"1.0"` (XML 1.1 exists but is rarely used)
- `encoding` — character encoding, typically `UTF-8` or `UTF-16`
- The prolog must appear **before** anything else in the document, including comments

### Elements

The fundamental building block of XML. An element has an opening tag, content, and a closing tag:

```xml
<name>John</name>
<age>28</age>
<isActive>true</isActive>
```

Content can be text, other elements, or a mix of both.

### Attributes

Metadata attached to an element's opening tag:

```xml
<person id="1" role="developer">
    <name>Long</name>
</person>
```

```ad-note
title: Elements vs Attributes — When to Use Which
There is no strict rule, but the convention is:
- **Elements** for data that the document is *about* (the content)
- **Attributes** for metadata *about* the element (IDs, types, formats)

A common guideline: if it could have sub-structure or if there could be multiple of them, use an element. If it's a simple modifier, use an attribute.
```

### Root Element Requirement

Every XML document must have **exactly one** root element that wraps all other content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<catalog>
    <book id="1">
        <title>C# in Depth</title>
        <author>Jon Skeet</author>
    </book>
    <book id="2">
        <title>CLR via C#</title>
        <author>Jeffrey Richter</author>
    </book>
</catalog>
```

Having two sibling root elements (e.g., `<book>...</book><book>...</book>` with no wrapper) is **invalid**.

### Nesting Rules

Elements must be **properly nested** — you cannot interleave tags:

```xml
<!-- VALID -->
<outer><inner>text</inner></outer>

<!-- INVALID — tags overlap -->
<outer><inner>text</outer></inner>
```

### Case Sensitivity

XML is **case-sensitive**. `<Name>`, `<name>`, and `<NAME>` are three different elements:

```xml
<Name>John</Name>   <!-- valid -->
<Name>John</name>   <!-- INVALID — mismatched case -->
```

### Comments

```xml
<!-- This is an XML comment -->
<!-- Comments cannot contain double dashes -- inside them -->
```

Comments cannot appear inside a tag: `<person <!-- bad -->>` is invalid.

### CDATA Sections

CDATA (Character Data) sections let you include raw text that the parser will **not** interpret as markup. Useful when your content contains characters like `<`, `>`, or `&`:

```xml
<script>
    <![CDATA[
        if (x < 10 && y > 5) {
            Console.WriteLine("Hello <World>");
        }
    ]]>
</script>
```

Without CDATA, you would need to escape: `&lt;`, `&gt;`, `&amp;`.

### Self-Closing Tags

Elements with no content can use a shorthand self-closing syntax:

```xml
<br/>
<image src="photo.png"/>

<!-- equivalent to -->
<br></br>
<image src="photo.png"></image>
```

---

## JSON vs XML Comparison

| Feature | JSON | XML |
|---|---|---|
| **Readability** | More concise, less noise | More verbose with opening/closing tags |
| **Data types** | Yes (string, number, bool, null, array, object) | No — everything is text |
| **Attributes** | No | Yes |
| **Comments** | No | Yes (`<!-- -->`) |
| **Schema validation** | JSON Schema | XSD, DTD, RelaxNG |
| **Namespaces** | No | Yes (for avoiding name collisions) |
| **Size** | Smaller | Larger (due to closing tags + verbosity) |
| **Parsing speed** | Generally faster | Generally slower |
| **Default in .NET** | `System.Text.Json` (since .NET Core 3.0) | `XmlSerializer`, `XDocument` |
| **Dominant use** | Web APIs, config, NoSQL | Enterprise/legacy systems, SOAP, document markup |

```ad-note
title: Which Should You Use?
**Default to JSON** for new projects — it's simpler, smaller, and the industry standard for APIs and configuration. Use **XML** when you need namespaces, attributes, mixed content (text + markup), or when interacting with systems that require it (SOAP services, legacy enterprise systems, Office XML formats).
```

---

## DTD (Document Type Definition)

A **DTD** defines the legal structure of an XML document — which elements are allowed, their order, nesting, and attributes. It is a way to **validate** that an XML document conforms to a specific structure.

### Internal DTD

Defined inside the XML document itself:

```xml
<?xml version="1.0"?>
<!DOCTYPE catalog [
    <!ELEMENT catalog (book+)>
    <!ELEMENT book (title, author, price)>
    <!ELEMENT title (#PCDATA)>
    <!ELEMENT author (#PCDATA)>
    <!ELEMENT price (#PCDATA)>
    <!ATTLIST book id CDATA #REQUIRED>
]>
<catalog>
    <book id="1">
        <title>C# in Depth</title>
        <author>Jon Skeet</author>
        <price>49.99</price>
    </book>
</catalog>
```

### External DTD

Referenced from a separate `.dtd` file:

```xml
<?xml version="1.0"?>
<!DOCTYPE catalog SYSTEM "catalog.dtd">
<catalog>
    <!-- ... -->
</catalog>
```

### Key DTD Declarations

| Declaration | Purpose | Example |
|---|---|---|
| `<!ELEMENT>` | Defines an element and its content | `<!ELEMENT title (#PCDATA)>` |
| `<!ATTLIST>` | Defines attributes for an element | `<!ATTLIST book id CDATA #REQUIRED>` |
| `<!ENTITY>` | Defines reusable text shortcuts | `<!ENTITY copyright "Copyright 2026">` |

- `#PCDATA` — parsed character data (text content)
- `+` — one or more, `*` — zero or more, `?` — zero or one
- `#REQUIRED` — attribute must be present, `#IMPLIED` — optional

```ad-note
title: DTD Is Largely Superseded
DTD is an older validation mechanism with significant limitations — it cannot define data types, does not support namespaces, and uses a non-XML syntax. **XML Schema (XSD)** has replaced DTD in most modern applications because it supports data types, namespaces, and is itself written in XML. However, you will encounter DTD in legacy XML documents and in HTML's `<!DOCTYPE html>` declaration.
```

---

## Related Topics

- [[Object Serialization Overview]]
- [[System.Text.Json Overview]]
- [[JsonSerializer]]
- [[JsonSerializerOptions]]
- [[XML Serialization]]
- [[Serialization Attributes]]
