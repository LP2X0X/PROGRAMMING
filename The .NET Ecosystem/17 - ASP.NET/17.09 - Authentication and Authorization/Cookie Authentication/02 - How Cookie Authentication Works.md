---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


The full cookie authentication flow involves a handshake between the browser, the server, and the authentication middleware. Here is the complete sequence:

### Step-by-Step Flow

**1. User submits login form**

The user navigates to a login page and submits their email/username and password via an HTML form. This is a standard HTTP POST request.

**2. Server validates credentials**

The server receives the POST request and checks the submitted credentials against a data store -- this could be a database table, [[ASP.NET Core Identity]], an external identity provider, or any custom validation logic.

**3. Server creates a ClaimsPrincipal**

If the credentials are valid, the server constructs a `ClaimsPrincipal` object. This object holds the user's **claims** -- pieces of information about the user such as their name, email, roles, and any custom data.

**4. Server calls `HttpContext.SignInAsync()`**

The server calls `HttpContext.SignInAsync()`, passing the `ClaimsPrincipal`. The cookie authentication handler takes over:
- It serializes the `ClaimsPrincipal` into bytes.
- It encrypts those bytes using the ASP.NET Core Data Protection system.
- It creates an HTTP `Set-Cookie` response header containing the encrypted payload.

**5. Browser stores the cookie**

The browser receives the `Set-Cookie` header and stores the cookie according to its attributes (domain, path, expiration, etc.).

**6. Subsequent requests include the cookie**

On every subsequent request to the same domain, the browser automatically attaches the authentication cookie in the `Cookie` request header. The user does not need to do anything -- this is built-in browser behavior.

**7. Middleware decrypts the cookie and sets `HttpContext.User`**

The cookie authentication middleware runs early in the ASP.NET Core request pipeline. It:
- Reads the cookie from the incoming request.
- Decrypts and deserializes it back into a `ClaimsPrincipal`.
- Sets `HttpContext.User` to that principal.

From this point onward, any code in the pipeline (controllers, Razor Pages, authorization filters) can access `HttpContext.User` to determine who the user is and what they are authorized to do.

> [!tip] Key Insight
> The cookie authentication middleware does **not** contact the database on every request. It reconstructs the user's identity entirely from the encrypted cookie. This makes cookie auth very fast -- but it also means that if the user's roles or permissions change in the database, the cookie will still contain the old data until it expires or the user re-logs in. You can use `OnValidatePrincipal` events to handle this (covered in the [[#Cookie Authentication Events]] section).

> [!summary] Section Summary
> The flow is: submit credentials, validate them, build a ClaimsPrincipal, encrypt it into a cookie via `SignInAsync()`, then on every subsequent request the middleware decrypts the cookie and populates `HttpContext.User`.
