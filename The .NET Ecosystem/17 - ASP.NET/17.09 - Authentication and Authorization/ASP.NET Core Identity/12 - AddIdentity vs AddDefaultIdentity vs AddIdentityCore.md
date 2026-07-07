---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


This is one of the most confusing aspects of Identity setup. ASP.NET Core offers three registration methods:

## AddIdentity<TUser, TRole>

```csharp
builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options => { ... })
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();
```

- Registers **full Identity** with user management AND role management
- Adds `UserManager<T>`, `SignInManager<T>`, and `RoleManager<T>` to DI
- Configures **cookie authentication** automatically
- Does **not** include default UI
- **Use when**: You want full Identity with roles and will build your own UI

## AddDefaultIdentity<TUser>

```csharp
builder.Services.AddDefaultIdentity<ApplicationUser>(options => { ... })
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

- Registers Identity with the **default Razor Pages UI**
- Adds `UserManager<T>` and `SignInManager<T>` to DI
- **Does not register `RoleManager<T>` by default** -- you must chain `.AddRoles<IdentityRole>()` if you need roles
- Configures cookie authentication automatically
- **Use when**: You want the built-in UI pages and may not need roles

```csharp
// AddDefaultIdentity WITH roles
builder.Services.AddDefaultIdentity<ApplicationUser>(options => { ... })
    .AddRoles<IdentityRole>()
    .AddEntityFrameworkStores<ApplicationDbContext>();
```

## AddIdentityCore<TUser>

```csharp
builder.Services.AddIdentityCore<ApplicationUser>(options => { ... })
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();
```

- Registers the **minimal core** of Identity -- just `UserManager<T>`
- Does **not** add `SignInManager<T>` or `RoleManager<T>`
- Does **not** configure any authentication scheme
- **Use when**: Building APIs where you handle authentication separately (e.g., JWT) and just need user/password management

## Comparison Table

| Feature | `AddIdentity` | `AddDefaultIdentity` | `AddIdentityCore` |
|---|---|---|---|
| `UserManager<T>` | Yes | Yes | Yes |
| `SignInManager<T>` | Yes | Yes | No |
| `RoleManager<T>` | Yes | No (unless `.AddRoles<T>()`) | No |
| Cookie auth configured | Yes | Yes | No |
| Default UI pages | No | Yes | No |
| Best for | MVC/Razor with custom UI | MVC/Razor with default UI | Web APIs |

> [!warning] Common Misconception
> Many tutorials use `AddDefaultIdentity` and then wonder why `RoleManager` is not available. If you need roles with `AddDefaultIdentity`, you must explicitly add `.AddRoles<IdentityRole>()`. With `AddIdentity`, roles are included by default because you specify the role type as a generic parameter.

> [!summary] Section Summary
> `AddIdentity<TUser, TRole>` gives you everything (users, roles, sign-in, cookies). `AddDefaultIdentity<TUser>` adds default UI but omits roles unless you chain `.AddRoles<T>()`. `AddIdentityCore<TUser>` is minimal -- just UserManager, no sign-in or auth scheme -- ideal for APIs.
