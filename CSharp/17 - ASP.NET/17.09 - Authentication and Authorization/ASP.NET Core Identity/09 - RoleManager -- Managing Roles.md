---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


> [!info] Definition
> **`RoleManager<TRole>`** provides methods for creating, deleting, and querying roles. Roles are named groups used to organize users for authorization purposes.

## Creating Roles

```csharp
public class RoleService
{
    private readonly RoleManager<IdentityRole> _roleManager;

    public RoleService(RoleManager<IdentityRole> roleManager)
    {
        _roleManager = roleManager;
    }

    public async Task EnsureRolesCreatedAsync()
    {
        string[] roles = { "Admin", "Manager", "Employee", "ReadOnly" };

        foreach (var role in roles)
        {
            if (!await _roleManager.RoleExistsAsync(role))
            {
                await _roleManager.CreateAsync(new IdentityRole(role));
            }
        }
    }
}
```

## Seeding Roles in Program.cs

A common pattern is to seed roles at application startup:

```csharp
// In Program.cs, after building the app
using (var scope = app.Services.CreateScope())
{
    var roleManager = scope.ServiceProvider
        .GetRequiredService<RoleManager<IdentityRole>>();
    var userManager = scope.ServiceProvider
        .GetRequiredService<UserManager<ApplicationUser>>();

    // Create roles
    string[] roles = { "Admin", "Manager", "Employee" };
    foreach (var role in roles)
    {
        if (!await roleManager.RoleExistsAsync(role))
        {
            await roleManager.CreateAsync(new IdentityRole(role));
        }
    }

    // Create a default admin user
    string adminEmail = "admin@company.com";
    var adminUser = await userManager.FindByEmailAsync(adminEmail);

    if (adminUser == null)
    {
        adminUser = new ApplicationUser
        {
            UserName = adminEmail,
            Email = adminEmail,
            FullName = "System Administrator",
            EmailConfirmed = true
        };

        await userManager.CreateAsync(adminUser, "Admin@123456");
        await userManager.AddToRoleAsync(adminUser, "Admin");
    }
}
```

> [!danger] Do Not Hard-Code Production Passwords
> The example above seeds a default admin password for development purposes. In production, use environment variables, Azure Key Vault, or another secrets management solution. Never commit real passwords to source control.

> [!summary] Section Summary
> `RoleManager<T>` manages role CRUD operations. Roles are typically seeded at application startup in `Program.cs` using a service scope. A common pattern is to create default roles and an initial admin user.
