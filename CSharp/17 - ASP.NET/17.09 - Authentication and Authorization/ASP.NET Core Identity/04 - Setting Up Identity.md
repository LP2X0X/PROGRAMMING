---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


## Service Registration

The core setup happens in `Program.cs`:

```csharp
builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    // Password requirements
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 8;
    options.Password.RequireNonAlphanumeric = true;
    options.Password.RequireUppercase = true;
    options.Password.RequireLowercase = true;
    options.Password.RequiredUniqueChars = 4;

    // Lockout settings
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
    options.Lockout.AllowedForNewUsers = true;

    // User settings
    options.User.RequireUniqueEmail = true;
    options.User.AllowedUserNameCharacters =
        "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789-._@+";

    // Sign-in settings
    options.SignIn.RequireConfirmedEmail = false;
    options.SignIn.RequireConfirmedAccount = false;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();
```

## Breaking Down Each Part

| Component | Purpose |
|---|---|
| `AddIdentity<ApplicationUser, IdentityRole>` | Registers Identity services with your custom user type and the built-in role type |
| `options.Password.*` | Controls password validation rules enforced during registration and password changes |
| `options.Lockout.*` | Configures account lockout behavior after failed login attempts |
| `options.User.*` | User-level constraints like unique email and allowed characters in usernames |
| `options.SignIn.*` | Controls whether confirmed email/phone/account is required to sign in |
| `.AddEntityFrameworkStores<T>()` | Registers EF Core as the backing store for Identity data |
| `.AddDefaultTokenProviders()` | Adds token providers for email confirmation, password reset, and 2FA |

## Adding the Authentication Middleware

After configuring services, you must add the middleware in the correct order:

```csharp
var app = builder.Build();

// ... other middleware

app.UseAuthentication();  // Must come before UseAuthorization
app.UseAuthorization();

// ... endpoint mapping
```

> [!danger] Middleware Order Matters
> `UseAuthentication()` **must** come before `UseAuthorization()`. If you reverse them, authorization will run before the user's identity is established, and every request will appear unauthenticated. This is one of the most common setup mistakes.

> [!summary] Section Summary
> Identity is configured via `AddIdentity<TUser, TRole>()` with options for passwords, lockout, user constraints, and sign-in requirements. It requires `.AddEntityFrameworkStores<T>()` for persistence and `.AddDefaultTokenProviders()` for token generation. The `UseAuthentication()` middleware must precede `UseAuthorization()`.
