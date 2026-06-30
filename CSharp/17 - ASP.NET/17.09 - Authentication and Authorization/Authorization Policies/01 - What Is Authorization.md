---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


> [!info] Definition
> **Authorization** is the process of determining whether an authenticated user has permission to access a specific resource or perform a specific action. It answers the question: "You are who you say you are -- but are you *allowed* to do this?"

Authorization is distinct from authentication. Authentication establishes *identity* -- who the user is. Authorization establishes *access* -- what the user can do. These two concepts work together but serve fundamentally different purposes.

| Concern          | Question Answered      | Example                          |
|---|---|---|
| Authentication   | Who are you?           | "I am user `john@example.com`"   |
| Authorization    | Are you allowed?       | "Can John access the admin panel?" |

In ASP.NET Core, authorization is handled by middleware that runs *after* authentication middleware. The authentication middleware establishes the user's identity (as a `ClaimsPrincipal`), and the authorization middleware evaluates policies against that identity to decide whether access is granted.

See [[Authentication Overview]] for details on how identity is established before authorization takes place.

> [!summary] Section Summary
> Authorization decides what an authenticated user can do. It runs after authentication in the middleware pipeline and evaluates policies against the user's claims and roles.
