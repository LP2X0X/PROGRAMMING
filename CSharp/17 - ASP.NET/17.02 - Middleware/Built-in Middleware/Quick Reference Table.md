---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## Quick Reference Table

| Middleware                  | Purpose                                                  | Required?                   | Typical Position                        | Key Gotcha                                                             |
|---|---|---|---|---|
| `UseExceptionHandler`       | Catches unhandled exceptions and returns error page/JSON | Yes (production)            | Very early (first or second)            | Must be before all middleware that can throw                           |
| `UseDeveloperExceptionPage` | Shows detailed stack trace page for developers           | No (dev only)               | Very first                              | **Never** enable in production -- exposes internals                    |
| `UseHsts`                   | Adds `Strict-Transport-Security` header                  | Recommended                 | Before HTTPS redirect                   | Do not use in development -- browsers cache the header                 |
| `UseHttpsRedirection`       | Redirects HTTP requests to HTTPS                         | Recommended                 | After HSTS, before static files         | Fails silently if HTTPS port is unknown                                |
| `UseStaticFiles`            | Serves files from `wwwroot` or custom paths              | If serving static content   | Before routing                          | Does not serve files outside configured paths by default               |
| `UseRouting`                | Matches request URL to an endpoint                       | Yes                         | After static files, before auth         | Must come before `UseAuthorization`                                    |
| `UseCors`                   | Adds CORS headers for cross-origin requests              | If API consumed by browsers | Between routing and auth                | Order matters -- must be after `UseRouting`, before `UseAuthorization` |
| `UseAuthentication`         | Runs authentication handler, sets `HttpContext.User`     | If using auth               | After routing, before authorization     | Does not reject unauthenticated requests on its own                    |
| `UseAuthorization`          | Enforces `[Authorize]` policies                          | If using auth               | After authentication                    | Requires `UseAuthentication` before it                                 |
| `UseSession`                | Enables session state via cookies                        | If using sessions           | After CORS, before endpoints            | Default session cookie is not essential -- GDPR implications           |
| `UseResponseCaching`        | Caches responses based on HTTP cache headers             | Optional performance        | After CORS, before endpoints            | Does not cache if `Authorization` header is present                    |
| `UseResponseCompression`    | Compresses response body with gzip/brotli                | Optional performance        | Very early (before static files)        | Do not use with HTTPS in security-sensitive scenarios (BREACH attack)  |
| `UseRateLimiter`            | Limits request rate per client/endpoint                  | Optional (.NET 7+)          | After routing, before auth or endpoints | Requires `.AddRateLimiter()` in service registration                   |

> [!summary] Section Summary
> The reference table above gives a bird's-eye view of each built-in middleware, its purpose, whether it is required, where it typically sits in the pipeline, and the single most important gotcha to watch for.
