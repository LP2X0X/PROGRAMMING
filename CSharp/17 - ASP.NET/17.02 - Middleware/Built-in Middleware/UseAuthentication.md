---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseAuthentication

**`UseAuthentication`** runs the configured **authentication handler** for each request. It examines the request (typically looking for a cookie, bearer token, or other credential) and populates `HttpContext.User` with a `ClaimsPrincipal` representing the authenticated identity.

### What Happens Internally

1. The middleware calls `IAuthenticationService.AuthenticateAsync()` using the **default authentication scheme**
2. The authentication handler (e.g., cookie handler, JWT bearer handler) inspects the request for credentials
3. If valid credentials are found, it creates a `ClaimsPrincipal` and assigns it to `HttpContext.User`
4. If no valid credentials are found, `HttpContext.User` is set to an anonymous/empty principal
5. The middleware **does not reject the request** -- it only identifies who the caller is

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.Authority = "https://auth.example.com";
        options.Audience = "order-api";
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            ValidateIssuer = true,
            ValidateAudience = true,
            ClockSkew = TimeSpan.FromMinutes(1)
        };
    });

// Program.cs -- middleware
app.UseAuthentication();
app.UseAuthorization();
```

### Multiple Authentication Schemes

```csharp
builder.Services.AddAuthentication()
    .AddJwtBearer("Bearer", options => { /* JWT config */ })
    .AddCookie("Cookies", options => { /* Cookie config */ });

// On a controller, specify which scheme:
[Authorize(AuthenticationSchemes = "Bearer")]
public class ApiOrdersController : ControllerBase { }

[Authorize(AuthenticationSchemes = "Cookies")]
public class WebOrdersController : Controller { }
```

### When You Need It

Any application that needs to know who the caller is, regardless of whether authorization is enforced.

### Gotchas

- `UseAuthentication` **does not deny access** to unauthenticated users -- that is the job of `UseAuthorization`
- Must be placed **before** `UseAuthorization` and **after** `UseRouting`
- If you forget to call `UseAuthentication`, `[Authorize]` attributes will still block requests, but `HttpContext.User` will never be populated -- leading to confusing 401 responses even with valid tokens

> [!summary] Section Summary
> `UseAuthentication` identifies the caller by running the configured authentication handler and populating `HttpContext.User`. It does not enforce access control -- that responsibility belongs to `UseAuthorization`. Always place it between `UseRouting` and `UseAuthorization`.
