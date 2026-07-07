---
tags:
  - csharp
  - asp-net-core
  - authentication
  - security
---


> [!info] Definition
> **Authentication** answers the question: *"Who are you?"*
> **Authorization** answers the question: *"What are you allowed to do?"*

These two concepts are often confused, but they are fundamentally different steps in the security pipeline. Authentication always comes first -- you cannot determine what someone is allowed to do until you know who they are.

**Analogy:** Imagine arriving at a concert venue. Authentication is showing your ID at the door -- proving you are who you claim to be. Authorization is the staff checking your ticket to see if it grants you access to the VIP section. You might be authenticated (they know your name) but not authorized (your ticket is general admission only).

| Aspect | Authentication | Authorization |
|---|---|---|
| Question | Who are you? | What can you do? |
| Happens | First | Second |
| HTTP Status on Failure | 401 Unauthorized | 403 Forbidden |
| Middleware | `UseAuthentication()` | `UseAuthorization()` |
| Namespace | `Microsoft.AspNetCore.Authentication` | `Microsoft.AspNetCore.Authorization` |

> [!warning] Common Misconception
> The HTTP status code `401` is named "Unauthorized," but it actually means **unauthenticated**. A `403 Forbidden` is the true "unauthorized" (authenticated but lacking permission). This naming inconsistency in the HTTP spec causes endless confusion.

> [!summary] Section Summary
> Authentication identifies the user. Authorization determines their permissions. They are separate middleware components in the ASP.NET Core pipeline, and authentication must always run before authorization.
