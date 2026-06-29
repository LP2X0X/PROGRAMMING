---
tags: [csharp, asp-net-core, project-structure]
---


The `Models/` folder contains classes that represent the data your application works with. These can be domain entities, DTOs (Data Transfer Objects), view models, or request/response objects.

### Typical Model Structure

```csharp
namespace OrderManagement.Models;

public class Order
{
    public int Id { get; set; }
    public string CustomerName { get; set; } = string.Empty;
    public DateTime OrderDate { get; set; }
    public decimal TotalAmount { get; set; }
    public OrderStatus Status { get; set; }
    public List<OrderItem> Items { get; set; } = new();
}

public class OrderItem
{
    public int Id { get; set; }
    public int OrderId { get; set; }
    public string ProductName { get; set; } = string.Empty;
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

public enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}
```

### View Models (MVC)

```csharp
namespace OrderManagement.Models;

public class OrderSummaryViewModel
{
    public int Id { get; set; }
    public string CustomerName { get; set; } = string.Empty;
    public DateTime OrderDate { get; set; }
    public decimal TotalAmount { get; set; }
    public int ItemCount { get; set; }
    public string StatusDisplay { get; set; } = string.Empty;
}
```

> [!tip] Models vs ViewModels vs DTOs
> - **Model / Entity** -- represents a database table or domain concept (e.g., `Order`, `Product`)
> - **ViewModel** -- shaped specifically for a view; may combine data from multiple entities
> - **DTO** -- used to transfer data across boundaries (API requests/responses); controls what data is exposed

> [!summary] Section Summary
> - `Models/` holds data classes: entities, view models, DTOs, enums
> - Entities map to database tables; view models are shaped for specific views
> - DTOs control the data shape for API request/response contracts
> - Keep models focused -- each class should represent one clear concept
