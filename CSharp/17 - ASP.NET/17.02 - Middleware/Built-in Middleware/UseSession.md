---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseSession

**`UseSession`** enables server-side session state, backed by a session store and identified by a session cookie sent to the client. Session data is stored server-side (in-memory by default, or in a distributed cache like Redis or SQL Server).

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddDistributedMemoryCache(); // Required: provides IDistributedCache
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(20);
    options.Cookie.Name = ".OrderPortal.Session";
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true; // Required for GDPR consent
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
    options.Cookie.SameSite = SameSiteMode.Strict;
});

// Program.cs -- middleware
app.UseSession();
```

### Using Session in Code

```csharp
// Setting session values
public IActionResult AddToCart(int productId)
{
    var cart = HttpContext.Session.GetString("ShoppingCart");
    var cartItems = cart != null 
        ? JsonSerializer.Deserialize<List<int>>(cart) 
        : new List<int>();

    cartItems.Add(productId);

    HttpContext.Session.SetString("ShoppingCart", 
        JsonSerializer.Serialize(cartItems));

    return RedirectToAction("ViewCart");
}

// Getting session values
public IActionResult ViewCart()
{
    var cart = HttpContext.Session.GetString("ShoppingCart");
    var cartItems = cart != null 
        ? JsonSerializer.Deserialize<List<int>>(cart) 
        : new List<int>();

    return View(cartItems);
}
```

### Distributed Session (Redis)

```csharp
builder.Services.AddStackExchangeRedisCache(options =>
{
    options.Configuration = "localhost:6379";
    options.InstanceName = "OrderPortal_";
});

builder.Services.AddSession(); // Now backed by Redis
```

### Configuration Options

| Option | Default | Description |
|---|---|---|
| `IdleTimeout` | 20 minutes | Session expires after this period of inactivity |
| `IOTimeout` | 1 minute | Maximum time for loading/committing session |
| `Cookie.Name` | `.AspNetCore.Session` | Name of the session ID cookie |
| `Cookie.HttpOnly` | `true` | Prevents JavaScript access to the cookie |
| `Cookie.IsEssential` | `false` | If `true`, session works even without GDPR consent |
| `Cookie.SecurePolicy` | `None` | Set to `Always` for HTTPS-only |

### When You Need It

When you need to store per-user state on the server between requests (shopping carts, wizard state, user preferences during a session).

### Gotchas

- `AddDistributedMemoryCache()` (or another `IDistributedCache`) **must** be registered before `AddSession()` -- otherwise you get a runtime exception
- The default in-memory cache is **not distributed** -- it does not work across multiple server instances. Use Redis or SQL Server for load-balanced deployments
- Session data is **not loaded automatically**. If you access `HttpContext.Session` without await `LoadAsync()`, it blocks synchronously. In middleware, always call `await HttpContext.Session.LoadAsync()` first
- Session cookies have GDPR implications. Set `IsEssential = true` only if the session is genuinely essential for the application to function

> [!summary] Section Summary
> `UseSession` provides server-side session storage identified by a cookie. Configure the backing store (`IDistributedCache`), cookie settings, and timeout. Use Redis or SQL Server for multi-server deployments. Be mindful of GDPR requirements for session cookies.
