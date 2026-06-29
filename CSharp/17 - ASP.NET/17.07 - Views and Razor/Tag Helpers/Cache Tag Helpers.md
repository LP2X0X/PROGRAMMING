---
tags:
  - csharp
  - asp-net-core
  - razor
  - tag-helpers
---


### In-Memory Cache Tag Helper

```cshtml
<cache expires-after="@TimeSpan.FromMinutes(10)">
    @* Expensive content rendered once, cached for 10 minutes *@
    @await Component.InvokeAsync("PopularProducts")
</cache>

<cache expires-sliding="@TimeSpan.FromMinutes(5)"
       vary-by-user="true"
       vary-by-query="category,page">
    @* Cached per user, varied by query string parameters *@
    <partial name="_RecommendedProducts" model="Model.Recommendations" />
</cache>
```

### Distributed Cache Tag Helper

For multi-server deployments:

```cshtml
<distributed-cache name="popular-products"
                   expires-after="@TimeSpan.FromMinutes(10)">
    @await Component.InvokeAsync("PopularProducts")
</distributed-cache>
```

Requires a distributed cache provider (Redis, SQL Server, etc.) registered in DI.

**Cache variation attributes:**

| Attribute | Varies cache by |
|---|---|
| `vary-by-user` | Authenticated user identity |
| `vary-by-query` | Query string parameters |
| `vary-by-route` | Route data values |
| `vary-by-cookie` | Cookie values |
| `vary-by-header` | Request headers |
| `vary-by` | Arbitrary string expression |

> [!summary] Section Summary
> - `<cache>` caches rendered HTML in memory with expiration policies
> - `<distributed-cache>` uses a distributed cache provider for multi-server scenarios
> - `vary-by-*` attributes create separate cache entries per user, query, route, etc.
> - Useful for expensive view components and partials that do not change frequently

---
