---
tags:
  - csharp
  - asp-net-core
  - middleware
  - pipeline
  - request
---

## Middleware Order Reference Table

| Position | Middleware | What It Does | Why This Position |
|---|---|---|---|
| 1 | `UseExceptionHandler()` / `UseDeveloperExceptionPage()` | Catches unhandled exceptions, returns error responses | Must wrap ALL other middleware to catch any exception from any component |
| 2 | `UseHsts()` | Adds `Strict-Transport-Security` header | Must run before content is served so the header appears on all responses |
| 3 | `UseHttpsRedirection()` | Redirects HTTP to HTTPS | Must run before any content (static or dynamic) is served over insecure HTTP |
| 4 | `UseStaticFiles()` | Serves CSS, JS, images from `wwwroot` | Before routing so static requests bypass the routing engine, auth, and other overhead |
| 5 | `UseRouting()` | Selects which endpoint matches the request URL | Before auth so that auth can read endpoint metadata (policies, attributes) |
| 6 | `UseCors()` | Handles CORS preflight and headers | After routing (needs endpoint metadata) but before auth (preflight has no credentials) |
| 7 | `UseAuthentication()` | Reads credentials, establishes `HttpContext.User` | After routing (may need endpoint-specific auth schemes), before authorization |
| 8 | `UseAuthorization()` | Checks user permissions against endpoint policies | After authentication (needs user identity) and after routing (needs endpoint metadata) |
| 9 | Custom middleware | Your application-specific logic | After all framework middleware so full context (user, endpoint, etc.) is available |
| 10 | `MapControllers()` / `MapRazorPages()` / `Map*()` | Executes the selected endpoint | Last -- all cross-cutting concerns must be resolved before the endpoint runs |

> [!summary] Section Summary
> The reference table provides a quick lookup for the correct position of each middleware and the reasoning behind it. Each position depends on the middleware before and after it.
