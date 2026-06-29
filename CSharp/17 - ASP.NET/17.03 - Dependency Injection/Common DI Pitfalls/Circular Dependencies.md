---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
---

## Circular Dependencies

> [!info] Definition
> A **circular dependency** occurs when Service A depends on Service B, and Service B depends on Service A (directly or through a chain). The DI container cannot resolve either service because each one requires the other to be constructed first.

### The Problem

```csharp
public class OrderService : IOrderService
{
    private readonly IInventoryService _inventoryService;

    public OrderService(IInventoryService inventoryService)
    {
        _inventoryService = inventoryService;
    }

    public void CreateOrder(Order order)
    {
        _inventoryService.ReserveStock(order.Items);
    }
}

public class InventoryService : IInventoryService
{
    private readonly IOrderService _orderService;

    public InventoryService(IOrderService orderService)
    {
        _orderService = orderService;
    }

    public void ReserveStock(List<OrderItem> items)
    {
        // Needs to check pending orders before reserving...
        var pendingOrders = _orderService.GetPendingOrders();
    }
}
```

### The Error

At runtime, when the container tries to resolve either service, you get a stack overflow or this exception:

```
System.InvalidOperationException: A circular dependency was detected for the service
of type 'IOrderService'.
IOrderService -> IInventoryService -> IOrderService
```

### Fix 1: Restructure to Eliminate the Circle

The best fix is to ask why the circular dependency exists and break it by extracting the shared logic:

```csharp
// Extract the shared concern into its own service
public interface IPendingOrderQuery
{
    List<Order> GetPendingOrders();
}

public class PendingOrderQuery : IPendingOrderQuery
{
    private readonly AppDbContext _dbContext;

    public PendingOrderQuery(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public List<Order> GetPendingOrders()
    {
        return _dbContext.Orders
            .Where(o => o.Status == OrderStatus.Pending)
            .ToList();
    }
}

// Now InventoryService depends on IPendingOrderQuery, not IOrderService
public class InventoryService : IInventoryService
{
    private readonly IPendingOrderQuery _pendingOrderQuery;

    public InventoryService(IPendingOrderQuery pendingOrderQuery)
    {
        _pendingOrderQuery = pendingOrderQuery;
    }

    public void ReserveStock(List<OrderItem> items)
    {
        var pendingOrders = _pendingOrderQuery.GetPendingOrders();
        // ... reservation logic
    }
}
```

### Fix 2: Use Lazy<T> to Break the Cycle

If restructuring is not immediately feasible, `Lazy<T>` defers resolution and breaks the cycle:

```csharp
// Registration
builder.Services.AddScoped<IOrderService, OrderService>();
builder.Services.AddScoped<IInventoryService, InventoryService>();
builder.Services.AddScoped(sp =>
    new Lazy<IOrderService>(() => sp.GetRequiredService<IOrderService>()));
```

```csharp
public class InventoryService : IInventoryService
{
    private readonly Lazy<IOrderService> _orderService;

    public InventoryService(Lazy<IOrderService> orderService)
    {
        _orderService = orderService;
    }

    public void ReserveStock(List<OrderItem> items)
    {
        // The IOrderService is only resolved when .Value is first accessed,
        // which is AFTER construction -- breaking the circular dependency.
        var pendingOrders = _orderService.Value.GetPendingOrders();
    }
}
```

> [!warning] Common Misconception
> `Lazy<T>` does not fix the underlying design problem -- it only defers it. The circular dependency still exists conceptually, and the code remains tightly coupled. Use `Lazy<T>` as a temporary bridge while you work on a proper restructuring.

### Fix 3: Introduce a Mediating Interface

Create an interface that both services can communicate through without depending on each other:

```csharp
public interface IStockReservationMediator
{
    void ReserveStock(List<OrderItem> items);
    List<Order> GetPendingOrders();
}

public class StockReservationMediator : IStockReservationMediator
{
    private readonly AppDbContext _dbContext;

    public StockReservationMediator(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public void ReserveStock(List<OrderItem> items)
    {
        // Direct database logic -- no dependency on either service
    }

    public List<Order> GetPendingOrders()
    {
        return _dbContext.Orders
            .Where(o => o.Status == OrderStatus.Pending)
            .ToList();
    }
}
```

> [!summary] Section Summary
> - Circular dependencies cause `InvalidOperationException` or stack overflow at resolution time
> - The best fix is restructuring: extract shared logic into a new service that breaks the cycle
> - `Lazy<T>` defers resolution and breaks the cycle technically, but the design problem remains
> - A mediating interface/service can sit between two services to eliminate the direct dependency
> - Circular dependencies are always a design smell -- they indicate responsibilities need to be redistributed
