---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


The following diagram shows how an HTTP request moves through the authorization pipeline:

```
Request
  |
  v
+---------------------------+
| Authentication Middleware |  --> Establishes ClaimsPrincipal (who is the user?)
+---------------------------+
  |
  v
+---------------------------+
| Authorization Middleware  |  --> Evaluates policies (is the user allowed?)
+---------------------------+
  |
  v
+---------------------------+
| Policy Evaluation         |  --> Finds the policy for the endpoint
| (which requirements?)     |
+---------------------------+
  |
  v
+---------------------------+
| Requirement Handlers      |  --> Each handler evaluates its requirement
| - Handler A: Succeed()    |
| - Handler B: (no action)  |
| - Handler C: Succeed()    |
+---------------------------+
  |
  v
+-------------------+     +-------------------+
| All requirements  | YES | Access Granted    |
| satisfied?        |---->| (200 OK)          |
+-------------------+     +-------------------+
  | NO
  v
+-------------------+     +-------------------+
| User              | YES | 401 Unauthorized  |
| authenticated?    | NO  | (redirect/reject) |
+-------------------+     +-------------------+
  | YES
  v
+-------------------+
| 403 Forbidden     |
| (access denied)   |
+-------------------+
```

> [!ad-note]
> The distinction between `401` and `403` is important:
> - **401 Unauthorized** -- the user is not authenticated. The response challenges the user to provide credentials.
> - **403 Forbidden** -- the user is authenticated but does not have permission. No amount of re-authenticating will help.

> [!summary] Section Summary
> Authorization flows through authentication middleware (identity), authorization middleware (policy lookup), and requirement handlers (evaluation). Unauthenticated users receive 401; authenticated but unauthorized users receive 403.
