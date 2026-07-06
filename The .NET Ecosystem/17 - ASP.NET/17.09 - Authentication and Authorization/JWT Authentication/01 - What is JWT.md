---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


> [!info] Definition
> **JSON Web Token (JWT)** is an open standard (RFC 7519) that defines a compact, URL-safe, self-contained way of transmitting information between parties as a JSON object. The information can be verified and trusted because it is digitally signed.

JWT is the dominant authentication mechanism for modern APIs. The key concept is **self-contained** -- the token itself carries all the information needed to identify and authorize the user. Unlike traditional session-based authentication where the server stores session data in memory or a database and the client carries only a session ID, a JWT embeds the user's identity and claims directly into the token.

This means:

- The server does **not** need to look up a session store on every request
- Any server in a load-balanced cluster can validate the token independently
- The token is portable across services and domains

> [!ad-note]
> "Self-contained" means the token is both the **ticket** and the **passenger manifest**. A session cookie is just a ticket number -- you have to go to the counter (the database) to find out who it belongs to. A JWT carries the passenger's name, seat assignment, and boarding group right on the ticket itself.

A JWT is typically sent in the `Authorization` header of an HTTP request:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

> [!summary] Section Summary
> JWT is a self-contained, stateless token format for API authentication. The token carries user identity and claims, eliminating the need for server-side session storage.
