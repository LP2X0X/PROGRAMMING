---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


Complex types are classes or records with multiple properties. The binder creates an instance and recursively sets each property by matching property names to request values.

### Basic Complex Type

```csharp
public class ProductSearchCriteria
{
    public string? Term { get; set; }
    public int Page { get; set; } = 1;
    public int PageSize { get; set; } = 20;
    public string SortBy { get; set; } = "name";
    public bool Descending { get; set; }
}

[HttpGet("products")]
public IActionResult Search(ProductSearchCriteria criteria)
{
    // GET /products?term=laptop&page=2&pageSize=50&sortBy=price&descending=true
    // 
    // criteria.Term = "laptop"
    // criteria.Page = 2
    // criteria.PageSize = 50
    // criteria.SortBy = "price"
    // criteria.Descending = true
}
```

### Property Name Matching

Property name matching is **case-insensitive**. The request value `pagesize`, `PageSize`, and `PAGESIZE` all match a property named `PageSize`.

### Nested Objects

Complex types can contain nested objects. The binder uses **dot notation** in form field names or query parameters:

```csharp
public class CreateCustomerRequest
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public Address ShippingAddress { get; set; } = new();
}

public class Address
{
    public string Street { get; set; } = string.Empty;
    public string City { get; set; } = string.Empty;
    public string State { get; set; } = string.Empty;
    public string ZipCode { get; set; } = string.Empty;
}

[HttpPost("customers")]
public IActionResult CreateCustomer(CreateCustomerRequest request)
{
    // Form fields or query parameters use dot notation:
    // Name=John+Doe
    // Email=john@example.com
    // ShippingAddress.Street=123+Main+St
    // ShippingAddress.City=Springfield
    // ShippingAddress.State=IL
    // ShippingAddress.ZipCode=62704
}
```

### Records

C# records work with model binding if they have a parameterless constructor or settable properties. The most compatible pattern uses `init` properties:

```csharp
public record ProductFilter
{
    public string? Term { get; init; }
    public decimal? MinPrice { get; init; }
    public decimal? MaxPrice { get; init; }
    public string? Category { get; init; }
}

[HttpGet("products")]
public IActionResult Search([FromQuery] ProductFilter filter)
{
    // GET /products?term=laptop&minPrice=500&maxPrice=1500&category=electronics
}
```

```ad-note
The binder needs to be able to create an instance of the type first, then set properties. Classes and records must have a **public parameterless constructor** (or the binder must be able to resolve one). Primary constructor records like `record ProductFilter(string Term, int Page)` can work but require all values to be present -- missing values cause binding failures rather than defaults.
```
