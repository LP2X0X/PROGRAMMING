---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


Cookie security is critical because the authentication cookie is the key to the user's session. If an attacker can steal or forge it, they can impersonate the user.

### HttpOnly

```csharp
options.Cookie.HttpOnly = true;  // This is the default
```

When `HttpOnly` is `true`, the cookie is inaccessible to JavaScript via `document.cookie`. This protects against **Cross-Site Scripting (XSS)** attacks. Even if an attacker injects malicious JavaScript into your page, that script cannot read or exfiltrate the authentication cookie.

> [!danger] Security Warning
> Never set `HttpOnly` to `false` for authentication cookies. There is no legitimate reason for client-side JavaScript to access the auth cookie. If you need user data on the client, expose it through a separate API endpoint instead.

### Secure

```csharp
options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
```

When set to `Always`, the cookie is only sent over HTTPS connections. This prevents the cookie from being transmitted in plaintext over HTTP, protecting against **man-in-the-middle (MITM)** attacks where an attacker on the same network could intercept the cookie.

| Value | Behavior |
|---|---|
| `Always` | Cookie only sent over HTTPS |
| `SameAsRequest` | Cookie sent over the same protocol as the request |
| `None` | Cookie sent over both HTTP and HTTPS |

### SameSite

```csharp
options.Cookie.SameSite = SameSiteMode.Strict;
```

The `SameSite` attribute controls whether the cookie is sent with cross-site requests, providing protection against **Cross-Site Request Forgery (CSRF)** attacks.

| Mode | Behavior | Use Case |
|---|---|---|
| `Strict` | Cookie never sent on cross-site requests | Maximum security; may break external links |
| `Lax` | Cookie sent on top-level navigations (GET) but not on cross-site POST | Good balance of security and usability |
| `None` | Cookie always sent (requires `Secure = true`) | Only for cross-site scenarios (OAuth, embedded iframes) |

> [!example] CSRF Attack Scenario
> Without `SameSite` protection, an attacker's website could include:
> ```html
> <form action="https://yourbank.com/transfer" method="POST">
>     <input type="hidden" name="amount" value="10000" />
>     <input type="hidden" name="to" value="attacker-account" />
> </form>
> <script>document.forms[0].submit();</script>
> ```
> When the victim visits the attacker's page, their browser would automatically attach the bank's auth cookie to this cross-site POST. With `SameSite = Strict` or `Lax`, the browser refuses to attach the cookie to this cross-origin request.

> [!tip] Recommended Security Configuration
> For most server-rendered applications, use this combination:
> ```csharp
> options.Cookie.HttpOnly = true;
> options.Cookie.SecurePolicy = CookieSecurePolicy.Always;
> options.Cookie.SameSite = SameSiteMode.Lax;
> ```
> Use `Strict` if your app is never linked to from external sites. Use `Lax` if users might follow links to your app from emails or other sites.

> [!summary] Section Summary
> `HttpOnly` prevents XSS cookie theft, `Secure` prevents MITM interception, and `SameSite` prevents CSRF attacks. Together, these three flags form the security foundation for authentication cookies.
