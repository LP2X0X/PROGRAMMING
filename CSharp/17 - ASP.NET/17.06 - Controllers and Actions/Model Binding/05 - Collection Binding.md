---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


Model binding supports arrays, lists, and dictionaries from both query strings and form data.

### Arrays and Lists from Query String

There are two syntaxes for binding collections from query strings:

**Repeated parameter names (preferred):**

```csharp
[HttpGet("products")]
public IActionResult GetByIds([FromQuery] List<int> ids)
{
    // GET /products?ids=1&ids=2&ids=3
    // ids = [1, 2, 3]
}
```

**Indexed syntax:**

```csharp
[HttpGet("products")]
public IActionResult GetByIds([FromQuery] List<int> ids)
{
    // GET /products?ids[0]=1&ids[1]=2&ids[2]=3
    // ids = [1, 2, 3]
    // Both syntaxes produce the same result
}
```

### Dictionaries from Query String

```csharp
[HttpGet("products")]
public IActionResult Search([FromQuery] Dictionary<string, string> filters)
{
    // GET /products?filters[category]=electronics&filters[brand]=acme&filters[inStock]=true
    //
    // filters = {
    //     ["category"] = "electronics",
    //     ["brand"] = "acme",
    //     ["inStock"] = "true"
    // }
}
```

### Lists of Complex Types from Form Data

Form data can bind collections of complex objects using indexed property names:

```csharp
public class OrderItemRequest
{
    public int ProductId { get; set; }
    public string ProductName { get; set; } = string.Empty;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

[HttpPost("orders")]
public IActionResult CreateOrder(List<OrderItemRequest> items)
{
    // Form data:
    // items[0].ProductId=1
    // items[0].ProductName=Widget
    // items[0].Quantity=3
    // items[0].UnitPrice=9.99
    // items[1].ProductId=2
    // items[1].ProductName=Gadget
    // items[1].Quantity=1
    // items[1].UnitPrice=24.99
    
    // items = [
    //     { ProductId=1, ProductName="Widget", Quantity=3, UnitPrice=9.99 },
    //     { ProductId=2, ProductName="Gadget", Quantity=1, UnitPrice=24.99 }
    // ]
}
```

```ad-warning
When binding lists of complex types, the indices **must be sequential starting from 0**. If you skip an index (e.g., `items[0]` then `items[2]`), the binder stops at the gap and only binds `items[0]`.
```
