---
tags:
  - csharp
  - asp-net-core
  - authorization
  - policies
  - security
---


Sometimes you need to conditionally show or hide UI elements based on authorization policies. You can inject `IAuthorizationService` into Razor views:

```csharp
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService

<nav>
    <a asp-action="Index" asp-controller="Home">Home</a>
    <a asp-action="Profile" asp-controller="Account">Profile</a>

    @if ((await AuthorizationService.AuthorizeAsync(User, "AdminPolicy")).Succeeded)
    {
        <a asp-action="Dashboard" asp-controller="Admin">Admin Panel</a>
    }

    @if ((await AuthorizationService.AuthorizeAsync(User, "ManagerPolicy")).Succeeded)
    {
        <a asp-action="Reports" asp-controller="Reports">Reports</a>
    }
</nav>
```

> [!warning] Common Misconception
> Hiding UI elements is **not** a security measure. It is a UX convenience. Users can still craft HTTP requests to hit endpoints directly. Always enforce authorization on the server side (controllers, handlers, middleware) in addition to hiding UI elements.

You can also check roles directly in Razor views without injecting `IAuthorizationService`:

```csharp
@if (User.IsInRole("Admin"))
{
    <a asp-action="ManageUsers" asp-controller="Admin">Manage Users</a>
}
```

> [!summary] Section Summary
> Inject `IAuthorizationService` into Razor views to conditionally render UI based on policies. Always enforce authorization on the server side -- hiding UI elements alone is not security.
