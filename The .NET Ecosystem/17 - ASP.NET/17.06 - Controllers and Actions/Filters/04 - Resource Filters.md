---
tags:
  - csharp
  - asp-net-core
  - filters
  - controllers
  - cross-cutting
---


Resource filters run **after authorization** but **before model binding**. They wrap the entire rest of the pipeline, so their "after" hook runs last.

- **Interface**: `IResourceFilter` / `IAsyncResourceFilter`
- Two hooks: `OnResourceExecuting` (before model binding) and `OnResourceExecuted` (after everything)
- Use cases: response caching, short-circuiting expensive operations, resource cleanup

### Caching Resource Filter Example

```csharp
public class CacheResourceFilter : IResourceFilter
{
    private readonly IMemoryCache _cache;

    public CacheResourceFilter(IMemoryCache cache)
    {
        _cache = cache;
    }

    public void OnResourceExecuting(ResourceExecutingContext context)
    {
        string cacheKey = context.HttpContext.Request.Path.ToString();

        if (_cache.TryGetValue(cacheKey, out IActionResult? cachedResult))
        {
            // Short-circuit: return cached result, skip the rest of the pipeline
            context.Result = cachedResult!;
        }
    }

    public void OnResourceExecuted(ResourceExecutedContext context)
    {
        if (context.Result is not null)
        {
            string cacheKey = context.HttpContext.Request.Path.ToString();

            var cacheOptions = new MemoryCacheEntryOptions()
                .SetAbsoluteExpiration(TimeSpan.FromMinutes(5));

            _cache.Set(cacheKey, context.Result, cacheOptions);
        }
    }
}
```

```ad-tip
Resource filters are ideal for caching because they wrap the entire pipeline. Setting `context.Result` in `OnResourceExecuting` short-circuits model binding, action execution, and everything else -- returning the cached response directly.
```
