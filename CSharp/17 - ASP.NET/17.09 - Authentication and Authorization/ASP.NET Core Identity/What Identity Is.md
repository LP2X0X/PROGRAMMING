---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


> [!info] Definition
> **ASP.NET Core Identity** is a full-featured, extensible user management system built into ASP.NET Core. It handles the entire lifecycle of user authentication and management -- from registration to password resets to two-factor authentication.

Identity is **not** just an authentication middleware. It is a complete framework that manages:

- **User accounts** -- creating, updating, deleting, and querying user records
- **Password management** -- hashing, validation rules, reset tokens
- **Roles and claims** -- grouping users and attaching fine-grained permissions
- **Token generation** -- email confirmation, password reset, two-factor codes
- **External login providers** -- integrating Google, Microsoft, Facebook, GitHub, and more
- **Account security** -- lockout policies, two-factor authentication, email confirmation

Identity sits on top of [[Entity Framework Core]] by default and stores all its data in a relational database. However, it is designed with abstraction layers (`IUserStore<T>`, `IRoleStore<T>`) that allow you to swap out the storage backend entirely.

> [!warning] Common Misconception
> Identity is often confused with [[Cookie Authentication]] or [[JWT Authentication]]. These are **authentication schemes** -- mechanisms for issuing and validating credentials. Identity is a **user management system** that can work *with* either of those schemes. You can use cookie auth without Identity, and you can use Identity without cookies (e.g., in an API that issues JWTs).

Think of Identity as the "back office" that manages who your users are, what their passwords are, and what permissions they have. The authentication scheme is the "front door" that validates incoming requests.

> [!summary] Section Summary
> ASP.NET Core Identity is a batteries-included user management framework. It handles users, passwords, roles, claims, tokens, 2FA, and external logins. It is not just authentication -- it is the entire identity management layer.
