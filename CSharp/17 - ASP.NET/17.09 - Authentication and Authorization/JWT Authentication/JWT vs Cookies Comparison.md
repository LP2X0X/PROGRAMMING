---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


| Aspect | JWT (Bearer Token) | Cookie-Based Sessions |
|---|---|---|
| **Use Case** | APIs, SPAs, mobile apps | Server-rendered web apps |
| **Storage** | Client-side (localStorage, memory, or cookie) | Server-side (memory, database, Redis) |
| **State** | Stateless -- server stores nothing | Stateful -- server maintains session store |
| **CSRF Vulnerability** | Not vulnerable (unless stored in cookies) | Vulnerable -- requires anti-forgery tokens |
| **XSS Vulnerability** | Vulnerable if in localStorage | Not vulnerable if `HttpOnly` cookie |
| **Cross-Domain** | Works naturally via `Authorization` header | Requires complex CORS cookie configuration |
| **Scalability** | Excellent -- no shared state needed | Requires sticky sessions or shared session store |
| **Revocation** | Difficult -- token valid until expiry | Easy -- delete server-side session |
| **Token Size** | Large (claims embedded in every request) | Small (just a session ID cookie) |
| **Server Load** | No database lookup per request | Database/cache lookup per request |

> [!ad-note]
> Neither approach is universally better. Choose based on your architecture. If you are building a monolithic server-rendered app, use cookies. If you are building an API consumed by multiple clients, use JWT.

> [!summary] Section Summary
> JWT and cookies serve different architectural needs. JWT excels in stateless, cross-domain, multi-client scenarios. Cookies excel in server-rendered apps where instant revocation and simpler security (HttpOnly, SameSite) matter.
