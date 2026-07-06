---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


JWT is an excellent choice in certain scenarios and a poor choice in others.

## Good Use Cases

| Scenario | Why JWT Works Well |
|---|---|
| SPAs (React, Angular, Vue) consuming APIs | Client stores the token, sends it with each request |
| Mobile applications | No cookie jar needed, tokens travel in headers |
| Microservices | Each service validates the token independently, no shared session store |
| Third-party API access | Tokens can be scoped and issued to external consumers |
| Cross-domain authentication | Tokens work across origins without CORS cookie issues |

## When NOT to Use JWT

| Scenario | Better Alternative |
|---|---|
| Server-rendered MVC/Razor Pages | [[Cookie Authentication]] with server-side sessions |
| Applications needing instant token revocation | Opaque tokens with a token store |
| Highly sensitive operations requiring per-request validation | Session-based auth with database lookup |

> [!warning] Common Misconception
> JWT is not inherently "more secure" than cookies. It is a different **mechanism** with different tradeoffs. Cookies with `HttpOnly`, `Secure`, and `SameSite` attributes are often more secure for browser-based applications because they are not accessible to JavaScript.

> [!ad-note]
> The primary advantage of JWT is **statelessness**. If your application runs on a single server and serves HTML pages, you gain nothing from JWT and lose the ability to revoke sessions instantly. Use cookies in that case.

> [!summary] Section Summary
> Use JWT for APIs consumed by SPAs, mobile apps, and microservices. Prefer cookie-based authentication for server-rendered applications. JWT shines when statelessness and cross-domain portability matter.
