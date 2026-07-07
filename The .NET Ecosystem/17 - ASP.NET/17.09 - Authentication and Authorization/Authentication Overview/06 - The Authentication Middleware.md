---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


The authentication middleware is registered by calling `app.UseAuthentication()` in your application's request pipeline. Its job is to inspect incoming requests, attempt to identify the caller, and populate `HttpContext.User` with the result.

### Middleware Ordering

The order in which middleware is registered matters enormously. The authentication middleware must sit **after routing** but **before authorization**:

```csharp
var app = builder.Build();

app.UseRouting();

app.UseAuthentication();  // WHO are you?
app.UseAuthorization();   // WHAT can you do?

app.MapControllers();
```

> [!danger] Critical
> If `UseAuthentication()` is placed after `UseAuthorization()`, authorization will run against an unauthenticated user every time. The `[Authorize]` attribute will never see a logged-in user, and every protected endpoint will return 401.

### What Happens When the Middleware Runs

When a request passes through the authentication middleware, the following occurs:

1. The middleware invokes the **default authentication scheme's handler**.
2. The handler inspects the request for credentials (a cookie, a JWT in the `Authorization` header, etc.).
3. If valid credentials are found, the handler creates a `ClaimsPrincipal` representing the user.
4. The middleware sets `HttpContext.User` to that principal.
5. If no credentials are found or they are invalid, `HttpContext.User` remains an anonymous (unauthenticated) principal.

> [!tip]
> The authentication middleware does **not** reject unauthenticated requests. It simply populates (or does not populate) the user. It is the **authorization** middleware that decides whether to allow or reject the request.

> [!summary] Section Summary
> The authentication middleware reads credentials from the request, validates them through a handler, and sets `HttpContext.User`. It must be placed after `UseRouting()` and before `UseAuthorization()` in the pipeline. It never rejects requests on its own.
