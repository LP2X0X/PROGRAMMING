---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


Real-world applications often need to support multiple authentication methods -- for example, cookies for browser-based users and JWT tokens for API clients.

### Configuration

```csharp
builder.Services.AddAuthentication(options =>
{
    // Default for browser requests
    options.DefaultScheme = "Cookies";
    options.DefaultChallengeScheme = "Cookies";
})
.AddCookie("Cookies", options =>
{
    options.LoginPath = "/Account/Login";
    options.ExpireTimeSpan = TimeSpan.FromHours(2);
})
.AddJwtBearer("Bearer", options =>
{
    options.Authority = "https://auth.example.com";
    options.Audience = "my-api";
});
```

### Selecting a Scheme Per Endpoint

Use the `[Authorize]` attribute to specify which scheme an endpoint should use:

```csharp
// Uses the default scheme (Cookies)
[Authorize]
public class DashboardController : Controller
{
    public IActionResult Index() => View();
}

// Explicitly uses JWT Bearer
[Authorize(AuthenticationSchemes = "Bearer")]
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    public IActionResult GetAll() => Ok(_products);
}

// Accepts EITHER cookies OR JWT
[Authorize(AuthenticationSchemes = "Cookies,Bearer")]
[Route("api/[controller]")]
public class HybridController : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok("Authenticated via either scheme");
}
```

### Policy-Based Scheme Selection

For more control, you can create authorization policies that specify schemes:

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("ApiPolicy", policy =>
    {
        policy.AuthenticationSchemes.Add("Bearer");
        policy.RequireAuthenticatedUser();
    });

    options.AddPolicy("WebPolicy", policy =>
    {
        policy.AuthenticationSchemes.Add("Cookies");
        policy.RequireAuthenticatedUser();
    });
});
```

```csharp
[Authorize(Policy = "ApiPolicy")]
public class SecureApiController : ControllerBase { }
```

> [!warning] Common Misconception
> When you specify `AuthenticationSchemes` on the `[Authorize]` attribute, **only** those schemes are used to authenticate the request for that endpoint. The default scheme is ignored. If the user authenticated via cookies but the endpoint requires `"Bearer"`, the user will be treated as unauthenticated for that endpoint.

> [!summary] Section Summary
> Multiple authentication schemes let you support different client types (browsers, APIs, mobile apps) simultaneously. Schemes can be selected per-endpoint using the `[Authorize]` attribute or via authorization policies. When schemes are specified explicitly, the default scheme is overridden for that endpoint.
