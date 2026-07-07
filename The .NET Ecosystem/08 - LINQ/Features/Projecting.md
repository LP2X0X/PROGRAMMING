---
tags:
 - csharp
 - linq
---

Projection means transforming each element in a sequence into a **different shape** — picking specific properties, computing new values, flattening nested collections, or reshaping into an entirely new type. In LINQ, projection is done with `Select` (or `select` in query syntax).

---

## Select — The Core Projection Operator

`Select` takes a lambda that receives each element and returns whatever you want. The result is an `IEnumerable<TResult>` where `TResult` is determined by what the lambda returns.

```csharp
string[] names = { "Alice", "Bob", "Carol" };

// Project to a different type
IEnumerable<int> lengths = names.Select(n => n.Length);
// { 5, 3, 5 }

// Project to a transformed version of the same type
IEnumerable<string> upper = names.Select(n => n.ToUpper());
// { "ALICE", "BOB", "CAROL" }
```

### Overload with Index

`Select` has an overload that passes the element's index as a second parameter:

```csharp
var numbered = names.Select((name, index) => $"{index + 1}. {name}");
// { "1. Alice", "2. Bob", "3. Carol" }
```

This is useful for numbering, alternating row styles, or any logic that depends on position.

---

## Projection Targets

### Into Anonymous Types

The most common projection pattern. Use `new { ... }` to create a lightweight, read-only object with only the properties you need. The compiler generates property names from the expressions you provide.

```csharp
// Method syntax
var nameDesc = products.Select(p => new { p.Name, p.Description });

// Query syntax
var nameDesc = from p in products
               select new { p.Name, p.Description };
```

Each element in `nameDesc` has `.Name` and `.Description` properties. Since the type is anonymous, you **must** use `var` — there is no type name to write.

You can also rename properties or compute new ones:

```csharp
var projected = products.Select(p => new
{
    ProductName = p.Name,
    DescriptionLength = p.Description.Length,
    IsExpensive = p.Price > 100
});
```

> [!note]
> Anonymous types generate value-based `Equals` and `GetHashCode`, so `Distinct()`, `GroupBy()`, and other equality-dependent operators work correctly on projections into anonymous types.

### Into Concrete Types

When you need to pass the results across method boundaries (anonymous types are scoped to the declaring method), project into a named type:

```csharp
// A simple DTO
public class ProductSummary
{
    public string Name { get; set; }
    public string Description { get; set; }
}

// Projection using object initializer syntax
IEnumerable<ProductSummary> summaries = products.Select(p => new ProductSummary
{
    Name = p.Name,
    Description = p.Description
});

// Query syntax equivalent
IEnumerable<ProductSummary> summaries =
    from p in products
    select new ProductSummary { Name = p.Name, Description = p.Description };
```

### Into Tuples

A middle ground between anonymous types and full classes. Tuples can be returned from methods, unlike anonymous types, and don't require a separate class definition:

```csharp
var nameAndPrice = products.Select(p => (p.Name, p.Price));

foreach (var item in nameAndPrice)
{
    Console.WriteLine($"{item.Name}: {item.Price:C}");
}

// With explicit tuple element names
var result = products.Select(p => (ProductName: p.Name, Cost: p.Price));
// Access via result.ProductName and result.Cost
```

### Into Scalar Values

Projecting to a single property or computed value:

```csharp
IEnumerable<string> names = products.Select(p => p.Name);
IEnumerable<decimal> taxes = products.Select(p => p.Price * 0.1m);
IEnumerable<bool> flags = products.Select(p => p.InStock);
```

---

## SelectMany — Flattening Nested Collections

`SelectMany` is the projection operator for **one-to-many** relationships. Where `Select` produces one output per input, `SelectMany` produces **zero or more** outputs per input and flattens them into a single sequence.

### The Problem SelectMany Solves

Consider a `Customer` with a list of `Orders`. Using `Select`:

```csharp
var ordersNested = customers.Select(c => c.Orders);
// Type: IEnumerable<List<Order>>
// Result: { [orderA, orderB], [orderC], [orderD, orderE, orderF] }
// This is a sequence of lists — nested, hard to work with
```

Using `SelectMany`:

```csharp
var ordersFlat = customers.SelectMany(c => c.Orders);
// Type: IEnumerable<Order>
// Result: { orderA, orderB, orderC, orderD, orderE, orderF }
// Flat sequence — each order is a top-level element
```

### SelectMany with Result Selector

An overload lets you combine the parent element with each flattened child:

```csharp
var result = customers.SelectMany(
    c => c.Orders,                           // collection selector
    (customer, order) => new                  // result selector
    {
        CustomerName = customer.Name,
        OrderId = order.Id,
        order.Total
    }
);
// Each result combines customer info with order info
```

### SelectMany in Query Syntax

In query syntax, `SelectMany` is expressed with **multiple `from` clauses**:

```csharp
var result = from customer in customers
             from order in customer.Orders
             select new { customer.Name, order.Id, order.Total };
```

This is exactly equivalent to the `SelectMany` with result selector above. The compiler transforms multiple `from` clauses into `SelectMany` calls.

### SelectMany with Index

Like `Select`, `SelectMany` has an overload that provides the index of the source element:

```csharp
var result = customers.SelectMany(
    (customer, index) => customer.Orders.Select(o => new
    {
        CustomerIndex = index,
        customer.Name,
        o.Id
    })
);
```

### Practical SelectMany Examples

#### Flatten a jagged array

```csharp
int[][] jagged = { new[] { 1, 2 }, new[] { 3, 4, 5 }, new[] { 6 } };

var flat = jagged.SelectMany(arr => arr);
// { 1, 2, 3, 4, 5, 6 }
```

#### Split strings into characters

```csharp
string[] words = { "hello", "world" };

var chars = words.SelectMany(w => w);
// { 'h', 'e', 'l', 'l', 'o', 'w', 'o', 'r', 'l', 'd' }
```

#### Split strings into words

```csharp
string[] sentences = { "hello world", "foo bar baz" };

var allWords = sentences.SelectMany(s => s.Split(' '));
// { "hello", "world", "foo", "bar", "baz" }
```

---

## Select vs SelectMany

| | `Select` | `SelectMany` |
|---|---|---|
| **Mapping** | One-to-one: each input → exactly one output | One-to-many: each input → zero or more outputs |
| **Return shape** | `IEnumerable<TResult>` | `IEnumerable<TResult>` (flattened) |
| **If lambda returns a collection** | You get a **nested** `IEnumerable<IEnumerable<T>>` | You get a **flat** `IEnumerable<T>` |
| **Query syntax** | `select` clause | Multiple `from` clauses |
| **SQL analogy** | `SELECT column` | `CROSS APPLY` / `INNER JOIN` on a sub-table |

---

## Projection and Deferred Execution

Both `Select` and `SelectMany` use [[Deferred Execution in LINQ|deferred execution]]. The lambda is not called when you write the query — it is called once per element when you enumerate the result. This means:

```csharp
var products = GetProducts();
var names = products.Select(p => p.Name); // no work done yet

products.Add(new Product { Name = "NewProduct" });

foreach (var name in names) // projection lambda runs here, includes "NewProduct"
{
    Console.WriteLine(name);
}
```

If you need a snapshot, materialize with `ToList()` or `ToArray()` — see [[Immediate Execution in LINQ]].
