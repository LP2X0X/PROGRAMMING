---
tags:
  - csharp
  - asp-net-core
  - razor
  - partial-views
  - view-components
---


View components support constructor injection, just like controllers. This is one of their primary advantages over partial views.

```csharp
public class RecentProductsViewComponent : ViewComponent
{
    private readonly IProductRepository _productRepo;
    private readonly IMemoryCache _cache;
    private readonly ILogger<RecentProductsViewComponent> _logger;

    public RecentProductsViewComponent(
        IProductRepository productRepo,
        IMemoryCache cache,
        ILogger<RecentProductsViewComponent> logger)
    {
        _productRepo = productRepo;
        _cache = cache;
        _logger = logger;
    }

    public async Task<IViewComponentResult> InvokeAsync(int count = 4)
    {
        var cacheKey = $"recent-products-{count}";

        if (!_cache.TryGetValue(cacheKey, out List<Product> products))
        {
            _logger.LogInformation("Cache miss for recent products");
            products = await _productRepo.GetRecentAsync(count);

            _cache.Set(cacheKey, products, TimeSpan.FromMinutes(5));
        }

        return View(products);
    }
}
```

Services must be registered in the DI container as usual:

```csharp
// Program.cs
builder.Services.AddScoped<IProductRepository, ProductRepository>();
builder.Services.AddMemoryCache();
```

> [!ad-note] Testability
> View components are straightforward to unit test because their dependencies are injected. Mock the services, call `InvokeAsync()`, and assert the returned `IViewComponentResult`. This is a significant advantage over embedding service calls in partial views via `@inject`.

> [!summary] Section Summary
> - View components support full constructor injection like controllers
> - Register dependencies in `Program.cs` as usual
> - This enables caching, logging, database access, and any service interaction
> - Constructor injection makes view components easy to unit test with mocked dependencies

---
