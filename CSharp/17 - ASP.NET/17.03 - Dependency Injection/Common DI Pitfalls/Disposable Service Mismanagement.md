---
tags:
  - csharp
  - asp-net-core
  - dependency-injection
  - pitfalls
---

## Disposable Service Mismanagement

The DI container manages the lifecycle of services it creates -- including calling `Dispose()` when appropriate. Misunderstanding this leads to either double-disposal bugs or resource leaks.

### Do Not Manually Dispose Injected Services

When the container creates a service, the container is responsible for disposing it. If you dispose it yourself, other consumers of that same instance get an `ObjectDisposedException`.

```csharp
// CustomerService.cs -- DANGEROUS MANUAL DISPOSAL
public class CustomerService : ICustomerService
{
    private readonly AppDbContext _dbContext;

    public CustomerService(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task<Customer> GetCustomerAsync(int id)
    {
        var customer = await _dbContext.Customers.FindAsync(id);

        // DO NOT DO THIS. The container will dispose the DbContext
        // when the scope ends. If you dispose it here, any other service
        // in the same scope that uses this DbContext will crash.
        _dbContext.Dispose(); // BAD -- causes ObjectDisposedException elsewhere
        
        return customer;
    }
}
```

> [!danger] Double Disposal
> If `CustomerService` disposes the `DbContext`, and then `OrderService` (in the same HTTP request scope) tries to use the same `DbContext` instance, it gets `ObjectDisposedException: Cannot access a disposed object`. The container will also try to dispose it again at scope end, causing a double-disposal scenario.

### When YOU Must Handle Disposal

If you create instances yourself (outside the container), the container does not know about them and will not dispose them:

```csharp
// FileExportService.cs -- YOU create it, YOU dispose it
public class FileExportService : IFileExportService
{
    public async Task ExportOrdersAsync(IEnumerable<Order> orders, string filePath)
    {
        // You created this StreamWriter with 'new' -- it is YOUR responsibility
        await using var writer = new StreamWriter(filePath);
        foreach (var order in orders)
        {
            await writer.WriteLineAsync($"{order.Id},{order.Total},{order.Date}");
        }
        // The 'await using' ensures disposal here -- correct
    }
}
```

Similarly, if you register a factory delegate that calls `new`:

```csharp
// Registration with a factory delegate
builder.Services.AddTransient<IReportWriter>(sp =>
{
    // The container calls this factory, but the instance created by 'new'
    // IS tracked by the container because it implements IDisposable.
    // The container WILL dispose it when the scope ends.
    return new CsvReportWriter(sp.GetRequiredService<IConfiguration>());
});
```

> [!ad-note]
> For services registered through the container (even via factory delegates), the container tracks `IDisposable` and `IAsyncDisposable` implementations and disposes them when their scope ends. The exception is Singleton services, which are disposed when the application shuts down (when the root `ServiceProvider` is disposed).

### IDisposable vs IAsyncDisposable

If your service implements `IAsyncDisposable`, the container will call `DisposeAsync()` instead of `Dispose()`. If it implements both, `DisposeAsync()` takes priority:

```csharp
public class OrderRepository : IOrderRepository, IAsyncDisposable
{
    private readonly DbConnection _connection;

    public OrderRepository(DbConnection connection)
    {
        _connection = connection;
    }

    public async ValueTask DisposeAsync()
    {
        // The container calls this automatically when the scope ends
        if (_connection.State == ConnectionState.Open)
        {
            await _connection.CloseAsync();
        }
        await _connection.DisposeAsync();
    }
}
```

> [!summary] Section Summary
> - Never call `Dispose()` on services that were injected by the container -- the container manages their lifecycle
> - The container disposes Scoped services at the end of the HTTP request, Singletons at application shutdown
> - If you create an object with `new` outside the container, you are responsible for disposing it
> - Factory-registered services are still tracked and disposed by the container
> - Prefer `IAsyncDisposable` over `IDisposable` for services with async cleanup (database connections, streams)
