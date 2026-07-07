---
tags:
  - csharp
  - serialization
  - object-graphs
---

## Object Graphs

When you serialize an object, the serializer does not just serialize that one object in isolation. It follows **all references** from that object and serializes the entire connected set of objects. This connected set is called the **object graph**.

Understanding object graphs is important because they determine **what actually gets serialized** — and where things can go wrong (circular references, unexpectedly large payloads, missing data).

---

## What Is an Object Graph?

An object graph is the complete network of objects reachable by following references from a starting (root) object. If object A has a field pointing to object B, and object B has a list containing objects C and D, then serializing A will serialize all four objects.

```ad-warning
title: Object Graph != Inheritance Hierarchy
The arrows in an object graph mean **"references"** (has-a / uses-a), not "inherits from" (is-a). An object graph is about runtime relationships between instances, not compile-time type relationships.
```

### Object Identifiers

During serialization, each object in the graph is assigned a **unique numerical identifier** so the serializer can:

1. **Track** which objects it has already visited
2. **Avoid duplicating** the same object if it is referenced multiple times
3. **Detect and handle** circular references instead of entering an infinite loop

---

## Simple Example

Consider these classes:

```csharp
public class Order
{
    public int Id { get; set; }
    public Customer Customer { get; set; }     // reference -> Customer is part of the graph
    public List<OrderItem> Items { get; set; }  // reference -> each OrderItem is part of the graph
}

public class Customer
{
    public string Name { get; set; }
}

public class OrderItem
{
    public string Product { get; set; }
    public int Qty { get; set; }
}
```

When you create and serialize an `Order`:

```csharp
var order = new Order
{
    Id = 1,
    Customer = new Customer { Name = "Long" },
    Items = new List<OrderItem>
    {
        new OrderItem { Product = "Widget", Qty = 2 },
        new OrderItem { Product = "Gadget", Qty = 1 }
    }
};

string json = JsonSerializer.Serialize(order);
```

The serializer traverses the entire object graph:

```
Order (#1)
├── Customer (#2) — "Long"
├── OrderItem (#3) — "Widget" x 2
└── OrderItem (#4) — "Gadget" x 1
```

**All 4 objects** are serialized, not just `Order`. The resulting JSON contains the customer and every order item because they are part of the graph.

```json
{
    "Id": 1,
    "Customer": { "Name": "Long" },
    "Items": [
        { "Product": "Widget", "Qty": 2 },
        { "Product": "Gadget", "Qty": 1 }
    ]
}
```

---

## Shared References

If two objects reference the **same** instance, the default behavior in most serializers is to serialize it **twice** (duplicating the data):

```csharp
var sharedCustomer = new Customer { Name = "Long" };

var order1 = new Order { Id = 1, Customer = sharedCustomer, Items = new() };
var order2 = new Order { Id = 2, Customer = sharedCustomer, Items = new() };

var orders = new List<Order> { order1, order2 };
// Default serialization: "Long" appears in both order objects (duplicated)
```

With `ReferenceHandler.Preserve`, the serializer uses `$id` / `$ref` metadata to avoid duplication:

```json
{
    "$id": "1",
    "$values": [
        {
            "$id": "2",
            "Id": 1,
            "Customer": { "$id": "3", "Name": "Long" },
            "Items": { "$id": "4", "$values": [] }
        },
        {
            "$id": "5",
            "Id": 2,
            "Customer": { "$ref": "3" },
            "Items": { "$id": "6", "$values": [] }
        }
    ]
}
```

The second order's `Customer` is `{ "$ref": "3" }` — a reference to the already-serialized customer, not a duplicate.

---

## Circular References

A **circular reference** occurs when object A references object B, which references back to object A (directly or through a chain). This creates an infinite loop for a naive serializer.

### Example

```csharp
public class Person
{
    public string Name { get; set; }
    public Person? BestFriend { get; set; }
}

var alice = new Person { Name = "Alice" };
var bob = new Person { Name = "Bob" };
alice.BestFriend = bob;
bob.BestFriend = alice;  // circular!
```

The object graph is:

```
Alice (#1) ──BestFriend──> Bob (#2)
  ^                          |
  └────────BestFriend────────┘
```

### Default Behavior: Exception

By default, `System.Text.Json`'s `JsonSerializer` **throws a `JsonException`** when it detects a circular reference. The default maximum depth is 64 levels, and it will throw once that depth is exceeded.

### Fix: ReferenceHandler

Configure `JsonSerializerOptions` to handle circular references:

```csharp
// Option 1: Preserve references with $id / $ref metadata
var preserveOptions = new JsonSerializerOptions
{
    ReferenceHandler = ReferenceHandler.Preserve
};
string json = JsonSerializer.Serialize(alice, preserveOptions);
// Produces $id and $ref tokens to break the cycle
```

```csharp
// Option 2: Ignore cycles by writing null where the cycle occurs
var ignoreOptions = new JsonSerializerOptions
{
    ReferenceHandler = ReferenceHandler.IgnoreCycles
};
string json = JsonSerializer.Serialize(alice, ignoreOptions);
// Bob's BestFriend will be null instead of referencing Alice
```

| ReferenceHandler | Behavior | Trade-off |
|---|---|---|
| `null` (default) | Throws `JsonException` on cycle | Safe — forces you to address the issue |
| `Preserve` | Adds `$id` / `$ref` metadata | Round-trips perfectly, but JSON is non-standard |
| `IgnoreCycles` | Writes `null` at the cycle point | Clean JSON, but loses the back-reference data |

```ad-note
title: When to Use Which
- Use **Preserve** when you need to serialize and deserialize the full graph with shared/circular references intact (e.g., internal data transfer between .NET services).
- Use **IgnoreCycles** when you need clean JSON for external consumers (APIs) and can tolerate losing the back-reference.
- Use **neither** (default) when your data model should not have cycles — the exception is a signal that something is wrong.
```

See [[JsonSerializerOptions]] for full details on configuring `ReferenceHandler` and other serialization options.

---

## Controlling the Graph Depth

Even without circular references, deeply nested object graphs can cause issues. `JsonSerializerOptions.MaxDepth` controls the maximum nesting level:

```csharp
var options = new JsonSerializerOptions
{
    MaxDepth = 32  // default is 64
};
```

If serialization exceeds this depth, a `JsonException` is thrown. If you are hitting this limit without circular references, it likely means your object model is excessively nested and may benefit from flattening.

---

## Excluding Objects from the Graph

Not every reference needs to be serialized. Use attributes to control what is included:

```csharp
public class Order
{
    public int Id { get; set; }

    [JsonIgnore]
    public Customer InternalCustomer { get; set; }  // excluded from serialization

    public string CustomerName => InternalCustomer?.Name;  // serialize this instead
}
```

See [[Serialization Attributes]] for `[JsonIgnore]`, `[JsonInclude]`, and other attributes that shape what enters the serialized graph.

---

## Related Topics

- [[Object Serialization Overview]]
- [[JsonSerializer]]
- [[JsonSerializerOptions]]
- [[Serialization Attributes]]
- [[JSON and XML Structure]]
- [[Custom JsonConverters]]
