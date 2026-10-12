---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
  - http
---


When a client sends data to an ASP.NET Core server, the data travels inside the HTTP request body in one of two formats: **form data** or **JSON**. These are fundamentally different wire formats, produced by different clients, and ASP.NET Core binds them through different mechanisms.

---

## The Two Formats on the Wire

### Form Data (URL-Encoded)

An HTML `<form>` with `method="post"` produces a request like this:

```http
POST /Products/Create HTTP/1.1
Host: localhost:5000
Content-Type: application/x-www-form-urlencoded

Name=Rice&Price=2.50&Category=Food
```

- The body is a flat list of **key-value pairs**, separated by `&`, with keys and values **URL-encoded** (spaces become `+` or `%20`, special characters become `%XX`)
- The browser generates this automatically when the user clicks Submit
- The `Content-Type` header is `application/x-www-form-urlencoded`
- For file uploads, the content type changes to `multipart/form-data`, which uses boundary delimiters instead of `&` to separate parts

```ad-info
URL encoding is necessary because characters like `&`, `=`, and spaces have structural meaning in the format. Without encoding, a product name like "Salt & Pepper" would break the parser -- the `&` would be interpreted as a field separator.
```

### JSON Body

A JavaScript frontend or mobile app typically sends data as JSON:

```http
POST /api/products HTTP/1.1
Host: localhost:5000
Content-Type: application/json

{"name":"Rice","price":2.5,"category":"Food"}
```

- The body is a single JSON object (or array) -- a structured, self-describing format
- The client constructs it explicitly using `JSON.stringify()` in JavaScript, or a serializer in mobile/desktop apps
- The `Content-Type` header is `application/json`
- JSON supports **nesting** and **arrays** natively, unlike form encoding

---

## Side-by-Side Comparison

| Aspect | Form Data | JSON Body |
|---|---|---|
| Wire format | `Name=Rice&Price=2.50` | `{"name":"Rice","price":2.5}` |
| Content-Type | `application/x-www-form-urlencoded` | `application/json` |
| Produced by | HTML `<form>` submit | `JSON.stringify()` / HTTP clients |
| Structure | Flat key-value pairs | Nested objects, arrays |
| Typical consumer | MVC / Razor Pages (server-rendered) | Web APIs (JS frontends, mobile apps) |
| ASP.NET binding | Default for MVC controllers | Requires `[FromBody]` or `[ApiController]` |
| Encoding overhead | URL-encoding (`%20`, `+`) | Minimal (UTF-8 text) |
| File uploads | Yes (`multipart/form-data`) | Not directly (use Base64 or multipart) |

---

## The Full Serialization / Deserialization Pipeline

Understanding the round trip clarifies where each conversion happens.

### JSON Round Trip

**Client to server:**

```
JS object  →  JSON.stringify()  →  JSON string  →  HTTP request body
    →  ASP.NET reads body  →  System.Text.Json deserializes  →  C# object
```

**Server to client:**

```
C# object  →  System.Text.Json serializes  →  JSON string  →  HTTP response body
    →  browser receives  →  response.json() / JSON.parse()  →  JS object
```

### Form Data Round Trip

**Client to server:**

```
HTML <form> inputs  →  browser encodes  →  key=value&key=value  →  HTTP request body
    →  ASP.NET FormValueProvider reads body  →  model binder maps to  →  C# object
```

There is no "server to client" equivalent for form data -- the server sends back HTML (for MVC) or JSON (for APIs), never URL-encoded form data.

```ad-tip
The JSON pipeline has a symmetric serialize/deserialize pair (`JSON.stringify` ↔ `JSON.parse` on the client, `JsonSerializer.Serialize` ↔ `JsonSerializer.Deserialize` on the server). Form data is asymmetric -- it only flows from client to server.
```

---

## Default Binding Behavior per Programming Model

Different ASP.NET Core programming models have different defaults for where they expect complex types to come from:

| Programming Model | Complex type binds from | Need `[FromBody]` for JSON? |
|---|---|---|
| MVC Controller (no `[ApiController]`) | Form values | **Yes** |
| Razor Pages | Form values | **Yes** |
| API Controller (`[ApiController]`) | Request body (JSON) | **No** -- inferred |
| Minimal APIs | Request body (JSON) | **No** -- inferred |

The reasoning behind these defaults:

- **MVC / Razor Pages** serve server-rendered HTML. Their clients are browsers submitting `<form>` elements, so form values are the natural default binding source. See [[02 - Binding Sources (Default Priority Order)]].
- **`[ApiController]`** controllers serve JavaScript frontends and mobile apps that communicate in JSON. The attribute flips the default so complex types bind from the request body (JSON) automatically. See [[11 - ApiController Changes to Default Behavior]].
- **Minimal APIs** follow the same logic as `[ApiController]` -- they are designed for API scenarios, so complex types bind from the body by default.

```ad-attention
This default difference is the single most common source of confusion for developers switching between MVC and API controllers. An action method that works perfectly in an `[ApiController]` will silently bind all properties to `null`/default in a plain MVC controller if the client sends JSON, because the MVC controller is looking for form values, not a JSON body.
```

---

## When to Use [FromBody] Explicitly

You need `[FromBody]` in two situations:

### 1. MVC Controller Accepting JSON

If you have an MVC controller (without `[ApiController]`) that needs to accept a JSON request body -- perhaps because a JavaScript snippet on the Razor page sends an AJAX request:

```csharp
// Without [FromBody] -- BROKEN for JSON requests
// ASP.NET looks for form values, finds none, all properties are null/default
[HttpPost]
public IActionResult Create(CreateProductRequest request)
{
    // request.Name == null, request.Price == 0
    // because no form values were sent
}

// With [FromBody] -- works with JSON requests
[HttpPost]
public IActionResult Create([FromBody] CreateProductRequest request)
{
    // request.Name == "Rice", request.Price == 2.5
    // System.Text.Json deserialized the JSON body
}
```

### 2. Clarifying Intent in Any Controller

Even in `[ApiController]` controllers where `[FromBody]` is inferred, some teams add it explicitly for readability:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    // [FromBody] is inferred -- this works without it
    [HttpPost]
    public IActionResult Create(CreateProductRequest request) { ... }
    
    // Explicit -- same behavior, but makes the binding source obvious
    [HttpPost]
    public IActionResult Create([FromBody] CreateProductRequest request) { ... }
}
```

```ad-info
You can also use `[FromForm]` on an `[ApiController]` action to override the default and accept form data instead of JSON. This is necessary when an API endpoint receives file uploads alongside other data. See [[05 - The ApiController Attribute]] for more on how `[ApiController]` changes defaults.
```

---

## How ASP.NET Chooses the Deserializer

ASP.NET Core uses **input formatters** to deserialize request bodies. The `Content-Type` header determines which formatter handles the request:

| Content-Type Header | Input Formatter | Result |
|---|---|---|
| `application/json` | `SystemTextJsonInputFormatter` | JSON deserialized via `System.Text.Json` |
| `application/x-www-form-urlencoded` | (Form value provider, not a formatter) | Key-value pairs parsed into form collection |
| `multipart/form-data` | (Form value provider) | Parts parsed into form collection + file collection |

```ad-warning
If a client sends JSON but forgets to set `Content-Type: application/json`, the request will fail with a **415 Unsupported Media Type** response. The JSON input formatter only activates when it sees the correct content type header. This is a common mistake when testing APIs with tools like `curl`.
```

---

## Nested and Complex Data

The two formats handle nesting very differently:

**JSON** -- nesting is natural:

```json
{
    "name": "Rice",
    "price": 2.5,
    "supplier": {
        "companyName": "FoodCorp",
        "contactEmail": "info@foodcorp.com"
    },
    "tags": ["grain", "staple"]
}
```

**Form data** -- nesting uses dot notation and bracket syntax:

```
Name=Rice&Price=2.50&Supplier.CompanyName=FoodCorp&Supplier.ContactEmail=info@foodcorp.com&Tags[0]=grain&Tags[1]=staple
```

Both will bind to the same C# model:

```csharp
public class CreateProductRequest
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public SupplierInfo Supplier { get; set; }
    public List<string> Tags { get; set; }
}
```

But the form-encoded version is harder to read, harder to construct manually, and more brittle with deeply nested structures. This is one reason APIs prefer JSON.

See [[04 - Complex Type Binding]] and [[05 - Collection Binding]] for the full binding rules for nested types and collections.

---

## Quick Reference: Which Format Is Being Sent?

When debugging binding issues, check these two things in the request:

1. **Content-Type header** -- tells ASP.NET which parser to use
2. **Raw body** -- tells you which format the client actually sent

| You see this Content-Type | Body looks like | Client is sending |
|---|---|---|
| `application/x-www-form-urlencoded` | `Key=Value&Key2=Value2` | Form data |
| `multipart/form-data; boundary=...` | `--boundary\r\nContent-Disposition...` | Form data (with files) |
| `application/json` | `{"key":"value"}` | JSON |
| *(missing)* | anything | Likely a bug -- ASP.NET may reject it |

```ad-tip
In development, use browser DevTools (Network tab → click request → Headers / Payload tabs) or a tool like Postman to inspect the raw request. This immediately tells you whether the client is sending form data or JSON, which determines whether `[FromBody]` is needed.
```
