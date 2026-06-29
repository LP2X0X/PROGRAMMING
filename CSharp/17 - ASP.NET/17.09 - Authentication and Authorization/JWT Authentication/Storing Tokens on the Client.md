---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


Where you store the JWT on the client side has significant security implications.

| Storage Method | XSS Vulnerable | CSRF Vulnerable | Persists Across Tabs | Persists After Close | Recommendation |
|---|---|---|---|---|---|
| `localStorage` | Yes | No | Yes | Yes | Acceptable for low-risk apps |
| `sessionStorage` | Yes | No | No (per-tab) | No | Slightly better than localStorage |
| HttpOnly Cookie | No | Yes | Yes | Configurable | Best for browser apps (add CSRF protection) |
| In-memory variable | No | No | No | No | Most secure, worst UX |

## localStorage

```javascript
// Store
localStorage.setItem('token', response.token);

// Retrieve and use
fetch('/api/data', {
    headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
});
```

> [!danger]
> `localStorage` is accessible to any JavaScript running on the page. A single XSS vulnerability means an attacker can steal every token stored there.

## HttpOnly Cookie (Recommended for SPAs)

The server sets the token as an HttpOnly cookie:

```csharp
Response.Cookies.Append("access_token", token, new CookieOptions
{
    HttpOnly = true,   // Not accessible to JavaScript
    Secure = true,     // Only sent over HTTPS
    SameSite = SameSiteMode.Strict,  // CSRF protection
    Expires = DateTimeOffset.UtcNow.AddMinutes(30)
});
```

> [!tip]
> The safest pattern for browser-based SPAs is: store the access token in an **HttpOnly cookie** and use `SameSite=Strict` or implement anti-CSRF tokens. This protects against both XSS and CSRF.

> [!summary] Section Summary
> Token storage is a security-critical decision. `localStorage` and `sessionStorage` are vulnerable to XSS. HttpOnly cookies are not accessible to JavaScript but require CSRF protection. For maximum security in browser apps, use HttpOnly cookies with `SameSite=Strict`.
