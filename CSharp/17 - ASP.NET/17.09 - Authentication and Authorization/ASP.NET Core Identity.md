---
tags: [csharp, asp-net-core, identity, authentication, security]
aliases: [Identity Framework, ASP.NET Identity, User Management System]
status: complete
date: 2026-06-18
---

# ASP.NET Core Identity

## Table of Contents

- [[#What Identity Is]]
- [[#What Identity Provides Out of the Box]]
- [[#Setting Up Identity]]
- [[#ApplicationUser -- Extending IdentityUser]]
- [[#Identity Database Tables]]
- [[#UserManager -- The Primary User Service]]
- [[#SignInManager -- Managing Sign-In]]
- [[#RoleManager -- Managing Roles]]
- [[#Identity with EF Core]]
- [[#Scaffolding Identity UI]]
- [[#Customizing Identity]]
- [[#Identity vs Manual Auth]]
- [[#AddIdentity vs AddDefaultIdentity vs AddIdentityCore]]
- [[#Complete Real-World Example]]
- [[#Comprehensive Summary]]
- [[#Related Topics]]
- [[#Further Reading]]

---

## What Identity Is

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

---

## What Identity Provides Out of the Box

Identity ships with a large surface area of functionality. Here is what you get without writing custom code:

### User Registration and Login

Identity provides `UserManager<T>` for creating user accounts with validated passwords and `SignInManager<T>` for authenticating users against stored credentials. It handles the full flow: validate input, hash the password, store the user, issue an authentication cookie.

### Password Hashing

> [!info] Definition
> **PBKDF2** (Password-Based Key Derivation Function 2) is the default password hashing algorithm used by Identity. In .NET 8+, it uses **600,000 iterations** with HMAC-SHA512, which is a significant increase from earlier versions.

Passwords are never stored in plain text. Identity hashes them using a one-way function and stores only the hash. When a user logs in, Identity hashes the provided password and compares it to the stored hash.

### Email Confirmation

Identity generates cryptographically secure tokens for email confirmation. After registration, you send the user an email with a confirmation link containing this token. When they click the link, Identity validates the token and marks their email as confirmed.

### Two-Factor Authentication (2FA)

Identity supports **TOTP-based** (Time-based One-Time Password) two-factor authentication, compatible with authenticator apps like Google Authenticator, Microsoft Authenticator, and Authy. Users scan a QR code, and the app generates rotating 6-digit codes.

### Account Lockout

After a configurable number of failed login attempts (default: 5), Identity locks the account for a configurable duration. This protects against brute-force attacks.

### Role Management

Roles are named groups that users can belong to. Identity provides `RoleManager<T>` for creating and managing roles, and supports checking role membership in authorization policies, `[Authorize(Roles = "Admin")]` attributes, and code-level checks.

### External Login Providers

Identity integrates with OAuth 2.0 and OpenID Connect providers out of the box. You can add Google, Microsoft, Facebook, GitHub, Twitter, and any generic OAuth provider. Identity handles the redirect flow, callback processing, and linking external accounts to local user records.

### Password Reset Tokens

When a user forgets their password, Identity generates a time-limited token. You send the token in a reset link, and Identity validates it before allowing the password change.

### Phone Number Confirmation

Similar to email confirmation, Identity can generate tokens to verify phone numbers via SMS. This is used for two-factor authentication via SMS (though TOTP is recommended over SMS for security).

> [!summary] Section Summary
> Identity provides user registration, login, PBKDF2 password hashing (600,000 iterations in .NET 8+), email confirmation, TOTP-based 2FA, account lockout, role management, external login providers (Google, Microsoft, etc.), password reset tokens, and phone number confirmation -- all out of the box.

---

## Setting Up Identity

### Service Registration

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

### Breaking Down Each Part

| Component | Purpose |
|-----------|---------|
| `AddIdentity<ApplicationUser, IdentityRole>` | Registers Identity services with your custom user type and the built-in role type |
| `options.Password.*` | Controls password validation rules enforced during registration and password changes |
| `options.Lockout.*` | Configures account lockout behavior after failed login attempts |
| `options.User.*` | User-level constraints like unique email and allowed characters in usernames |
| `options.SignIn.*` | Controls whether confirmed email/phone/account is required to sign in |
| `.AddEntityFrameworkStores<T>()` | Registers EF Core as the backing store for Identity data |
| `.AddDefaultTokenProviders()` | Adds token providers for email confirmation, password reset, and 2FA |

### Adding the Authentication Middleware

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

---

## ApplicationUser -- Extending IdentityUser

### What IdentityUser Already Provides

The base `IdentityUser` class (from `Microsoft.AspNetCore.Identity`) comes with these properties:

| Property | Type | Description |
|----------|------|-------------|
| `Id` | `string` | Unique identifier (GUID by default) |
| `UserName` | `string` | The username |
| `NormalizedUserName` | `string` | Uppercase version for case-insensitive lookups |
| `Email` | `string` | Email address |
| `NormalizedEmail` | `string` | Uppercase email for lookups |
| `EmailConfirmed` | `bool` | Whether email is verified |
| `PasswordHash` | `string` | The hashed password |
| `SecurityStamp` | `string` | Changes when credentials change (invalidates old tokens) |
| `ConcurrencyStamp` | `string` | Optimistic concurrency token |
| `PhoneNumber` | `string` | Phone number |
| `PhoneNumberConfirmed` | `bool` | Whether phone is verified |
| `TwoFactorEnabled` | `bool` | Whether 2FA is enabled |
| `LockoutEnd` | `DateTimeOffset?` | When the lockout expires |
| `LockoutEnabled` | `bool` | Whether lockout is allowed |
| `AccessFailedCount` | `int` | Count of failed login attempts |

### Creating a Custom User Class

You almost always want to extend `IdentityUser` with application-specific properties:

```csharp
public class ApplicationUser : IdentityUser
{
    [Required]
    [MaxLength(100)]
    public string FullName { get; set; } = string.Empty;

    [MaxLength(50)]
    public string Department { get; set; } = string.Empty;

    public DateTime HireDate { get; set; }

    public string? ProfilePictureUrl { get; set; }

    public bool IsActive { get; set; } = true;
}
```

> [!tip] Use ApplicationUser Everywhere
> Once you create `ApplicationUser`, use it consistently across your entire application -- in `AddIdentity<ApplicationUser, ...>`, in `UserManager<ApplicationUser>`, in `SignInManager<ApplicationUser>`, and in your `IdentityDbContext<ApplicationUser>`. Mixing up the generic type parameter is a common source of DI resolution errors.

> [!warning] Common Misconception
> The `Id` property on `IdentityUser` is a `string` by default, not an `int` or `Guid`. It stores a GUID as a string. If you want a different key type, you can inherit from `IdentityUser<TKey>` instead, e.g., `IdentityUser<int>` or `IdentityUser<Guid>`. But be aware this changes the generic parameters across the entire Identity stack.

> [!summary] Section Summary
> `IdentityUser` provides ~15 built-in properties including Id, UserName, Email, PasswordHash, and security-related fields. Extend it via `ApplicationUser : IdentityUser` to add domain-specific properties like Department and HireDate. Use `ApplicationUser` consistently in all Identity generics.

---

## Identity Database Tables

When you run EF Core migrations with Identity, the following tables are created:

| Table | Purpose | Key Columns |
|-------|---------|-------------|
| **AspNetUsers** | Stores user accounts | Id, UserName, Email, PasswordHash, SecurityStamp, plus your custom columns |
| **AspNetRoles** | Defines available roles | Id, Name, NormalizedName |
| **AspNetUserRoles** | Maps users to roles (many-to-many) | UserId, RoleId |
| **AspNetUserClaims** | Stores claims attached directly to users | Id, UserId, ClaimType, ClaimValue |
| **AspNetRoleClaims** | Stores claims attached to roles (inherited by role members) | Id, RoleId, ClaimType, ClaimValue |
| **AspNetUserLogins** | Associates external login providers with user accounts | LoginProvider, ProviderKey, UserId |
| **AspNetUserTokens** | Stores tokens (2FA recovery codes, authenticator keys) | UserId, LoginProvider, Name, Value |

### How the Tables Relate

```
AspNetUsers (1) ----< (many) AspNetUserRoles (many) >---- (1) AspNetRoles
     |                                                          |
     |---< AspNetUserClaims                                     |---< AspNetRoleClaims
     |---< AspNetUserLogins
     |---< AspNetUserTokens
```

> [!tip] Claims From Roles
> When a user belongs to a role, they automatically inherit that role's claims from `AspNetRoleClaims`. This is why role claims are powerful -- you can define a set of permissions on a role and every user in that role gets them without individual claim assignments.

> [!ad-note] On the SecurityStamp Column
> `SecurityStamp` in `AspNetUsers` is a critical security feature. It changes whenever a user's credentials change (password, email, 2FA settings). Existing authentication cookies are validated against this stamp, so changing your password immediately invalidates all your other sessions.

> [!summary] Section Summary
> Identity creates seven tables: AspNetUsers, AspNetRoles, AspNetUserRoles, AspNetUserClaims, AspNetRoleClaims, AspNetUserLogins, and AspNetUserTokens. Together, they store the complete user management data model including accounts, roles, claims, external logins, and tokens.

---

## UserManager -- The Primary User Service

> [!info] Definition
> **`UserManager<TUser>`** is the primary service for performing CRUD operations on user accounts. It is injected via DI and provides methods for creating users, managing passwords, handling claims, generating tokens, and more.

### Common Operations

#### Creating a User

```csharp
public class AccountService
{
    private readonly UserManager<ApplicationUser> _userManager;

    public AccountService(UserManager<ApplicationUser> userManager)
    {
        _userManager = userManager;
    }

    public async Task<IdentityResult> RegisterUserAsync(
        string email, string password, string fullName)
    {
        var user = new ApplicationUser
        {
            UserName = email,
            Email = email,
            FullName = fullName,
            HireDate = DateTime.UtcNow
        };

        // CreateAsync hashes the password and saves the user
        IdentityResult result = await _userManager.CreateAsync(user, password);

        if (result.Succeeded)
        {
            // Optionally assign a default role
            await _userManager.AddToRoleAsync(user, "Employee");
        }

        return result;
    }
}
```

> [!warning] Common Misconception
> `CreateAsync` returns an `IdentityResult`, not the user. You must check `result.Succeeded` before proceeding. If it fails (e.g., password too weak, duplicate email), the errors are in `result.Errors` -- a collection of `IdentityError` objects with `Code` and `Description` properties.

#### Finding Users

```csharp
// By email
ApplicationUser? user = await _userManager.FindByEmailAsync("john@example.com");

// By ID
ApplicationUser? user = await _userManager.FindByIdAsync("some-guid-string");

// By username
ApplicationUser? user = await _userManager.FindByNameAsync("john@example.com");
```

#### Password Operations

```csharp
// Verify a password without signing in
bool isValid = await _userManager.CheckPasswordAsync(user, "MyPassword123!");

// Change password (requires old password)
IdentityResult result = await _userManager.ChangePasswordAsync(
    user, "OldPassword", "NewPassword123!");

// Reset password (uses a token -- for "forgot password" flows)
string token = await _userManager.GeneratePasswordResetTokenAsync(user);
// ... send token to user via email ...
IdentityResult result = await _userManager.ResetPasswordAsync(
    user, token, "NewPassword123!");
```

#### Claims Management

```csharp
// Add a claim to a user
await _userManager.AddClaimAsync(user, new Claim("Department", "Engineering"));

// Get all claims for a user
IList<Claim> claims = await _userManager.GetClaimsAsync(user);

// Remove a claim
await _userManager.RemoveClaimAsync(user, existingClaim);
```

#### Email Confirmation

```csharp
// Generate confirmation token
string token = await _userManager.GenerateEmailConfirmationTokenAsync(user);
// ... send email with token ...

// Confirm the email when user clicks the link
IdentityResult result = await _userManager.ConfirmEmailAsync(user, token);

// Check if email is confirmed
bool confirmed = await _userManager.IsEmailConfirmedAsync(user);
```

### Key UserManager Methods Reference

| Method | Purpose |
|--------|---------|
| `CreateAsync(user, password)` | Create a user with a hashed password |
| `DeleteAsync(user)` | Delete a user |
| `UpdateAsync(user)` | Save changes to a user |
| `FindByEmailAsync(email)` | Find user by email |
| `FindByIdAsync(id)` | Find user by ID |
| `CheckPasswordAsync(user, password)` | Verify a password |
| `AddToRoleAsync(user, role)` | Assign a role |
| `RemoveFromRoleAsync(user, role)` | Remove a role |
| `GetRolesAsync(user)` | Get user's roles |
| `IsInRoleAsync(user, role)` | Check role membership |
| `AddClaimAsync(user, claim)` | Add a claim |
| `GetClaimsAsync(user)` | Get claims |
| `GenerateEmailConfirmationTokenAsync(user)` | Token for email confirmation |
| `ConfirmEmailAsync(user, token)` | Confirm email |
| `GeneratePasswordResetTokenAsync(user)` | Token for password reset |
| `ResetPasswordAsync(user, token, newPassword)` | Reset password with token |

> [!summary] Section Summary
> `UserManager<T>` is the central service for all user operations. It provides methods for CRUD, password management, role assignment, claims management, and token generation. Always check `IdentityResult.Succeeded` after mutating operations.

---

## SignInManager -- Managing Sign-In

> [!info] Definition
> **`SignInManager<TUser>`** handles the sign-in and sign-out process. It works with the authentication middleware to issue and revoke authentication cookies.

### Password Sign-In

```csharp
public class AuthController : Controller
{
    private readonly SignInManager<ApplicationUser> _signInManager;
    private readonly UserManager<ApplicationUser> _userManager;

    public AuthController(
        SignInManager<ApplicationUser> signInManager,
        UserManager<ApplicationUser> userManager)
    {
        _signInManager = signInManager;
        _userManager = userManager;
    }

    [HttpPost]
    public async Task<IActionResult> Login(LoginViewModel model)
    {
        if (!ModelState.IsValid)
            return View(model);

        var result = await _signInManager.PasswordSignInAsync(
            model.Email,
            model.Password,
            isPersistent: model.RememberMe,   // persistent cookie
            lockoutOnFailure: true             // increment failed count
        );

        if (result.Succeeded)
            return RedirectToAction("Index", "Home");

        if (result.IsLockedOut)
            return View("Lockout");

        if (result.RequiresTwoFactor)
            return RedirectToAction("LoginWith2fa");

        if (result.IsNotAllowed)
        {
            // Email not confirmed, etc.
            ModelState.AddModelError("", "Login not allowed. Confirm your email first.");
            return View(model);
        }

        ModelState.AddModelError("", "Invalid login attempt.");
        return View(model);
    }
}
```

### Sign-In Result Properties

| Property | Meaning |
|----------|---------|
| `Succeeded` | Login was successful |
| `IsLockedOut` | Account is currently locked out |
| `RequiresTwoFactor` | User has 2FA enabled, needs second factor |
| `IsNotAllowed` | Sign-in not allowed (e.g., unconfirmed email when `RequireConfirmedEmail = true`) |

### Signing Out

```csharp
[HttpPost]
public async Task<IActionResult> Logout()
{
    await _signInManager.SignOutAsync();
    return RedirectToAction("Index", "Home");
}
```

### Checking If a User Is Signed In

```csharp
// In a controller or Razor Page
if (_signInManager.IsSignedIn(User))
{
    // User is authenticated
}

// In a Razor view
@if (SignInManager.IsSignedIn(User))
{
    <a asp-action="Logout">Log Out</a>
}
```

### External Login Flow

```csharp
// Step 1: Challenge -- redirect to external provider
[HttpPost]
public IActionResult ExternalLogin(string provider)
{
    var redirectUrl = Url.Action("ExternalLoginCallback");
    var properties = _signInManager.ConfigureExternalAuthenticationProperties(
        provider, redirectUrl);
    return Challenge(properties, provider);
}

// Step 2: Callback -- handle the response
[HttpGet]
public async Task<IActionResult> ExternalLoginCallback()
{
    var info = await _signInManager.GetExternalLoginInfoAsync();
    if (info == null)
        return RedirectToAction("Login");

    // Try to sign in with the external login
    var result = await _signInManager.ExternalLoginSignInAsync(
        info.LoginProvider, info.ProviderKey, isPersistent: false);

    if (result.Succeeded)
        return RedirectToAction("Index", "Home");

    // If user doesn't exist yet, create an account
    var email = info.Principal.FindFirstValue(ClaimTypes.Email);
    // ... create user and link external login ...
}
```

> [!summary] Section Summary
> `SignInManager<T>` handles authentication flows -- password sign-in, sign-out, external logins, and 2FA. `PasswordSignInAsync` returns a rich result object indicating success, lockout, 2FA requirement, or denial. Always check all result properties to handle each scenario.

---

## RoleManager -- Managing Roles

> [!info] Definition
> **`RoleManager<TRole>`** provides methods for creating, deleting, and querying roles. Roles are named groups used to organize users for authorization purposes.

### Creating Roles

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

### Seeding Roles in Program.cs

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

---

## Identity with EF Core

### The DbContext

Identity requires a DbContext that inherits from `IdentityDbContext<TUser>`:

```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    // Your application's own entities
    public DbSet<Product> Products { get; set; }
    public DbSet<Order> Orders { get; set; }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        // IMPORTANT: Always call base -- it configures Identity tables
        base.OnModelCreating(builder);

        // Optional: customize Identity table names
        builder.Entity<ApplicationUser>().ToTable("Users");
        builder.Entity<IdentityRole>().ToTable("Roles");
        builder.Entity<IdentityUserRole<string>>().ToTable("UserRoles");
        builder.Entity<IdentityUserClaim<string>>().ToTable("UserClaims");
        builder.Entity<IdentityRoleClaim<string>>().ToTable("RoleClaims");
        builder.Entity<IdentityUserLogin<string>>().ToTable("UserLogins");
        builder.Entity<IdentityUserToken<string>>().ToTable("UserTokens");

        // Your own entity configurations
        builder.Entity<Product>(entity =>
        {
            entity.HasKey(p => p.Id);
            // ...
        });
    }
}
```

> [!danger] Always Call base.OnModelCreating
> Forgetting `base.OnModelCreating(builder)` is a guaranteed runtime error. The base class configures all the Identity table relationships, keys, and indexes. Without it, EF Core will not know how to create or query the Identity tables.

### Registering the DbContext

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));
```

### Running Migrations

```bash
# Create the initial migration
dotnet ef migrations add CreateIdentitySchema

# Apply the migration to the database
dotnet ef database update

# If you add custom properties to ApplicationUser later
dotnet ef migrations add AddDepartmentToUsers
dotnet ef database update
```

> [!tip] Separate Migration Projects
> In larger solutions, you may want your migrations in a separate class library project. Use `dotnet ef migrations add ... --project MyApp.Data --startup-project MyApp.Web` to manage this.

> [!summary] Section Summary
> Identity uses EF Core through `IdentityDbContext<TUser>`. Always call `base.OnModelCreating()` in your override. You can customize table names and add your own `DbSet` properties alongside Identity's tables. Use standard EF Core migration commands to create and update the database schema.

---

## Scaffolding Identity UI

ASP.NET Core can generate the complete set of Identity Razor Pages for you:

```bash
# Install the code generator tool
dotnet tool install -g dotnet-aspnet-codegenerator

# Scaffold all Identity pages
dotnet aspnet-codegenerator identity --dbContext ApplicationDbContext

# Scaffold specific pages only
dotnet aspnet-codegenerator identity --dbContext ApplicationDbContext \
    --files "Account.Login;Account.Register;Account.Logout;Account.ForgotPassword"
```

### What Gets Generated

Scaffolding creates a folder structure under `Areas/Identity/Pages/Account/`:

| Page | Purpose |
|------|---------|
| `Register.cshtml` | User registration form |
| `Login.cshtml` | Login form |
| `Logout.cshtml` | Logout confirmation |
| `ForgotPassword.cshtml` | Initiate password reset |
| `ResetPassword.cshtml` | Enter new password with token |
| `ConfirmEmail.cshtml` | Email confirmation landing page |
| `Manage/Index.cshtml` | User profile management |
| `Manage/ChangePassword.cshtml` | Change password |
| `Manage/TwoFactorAuthentication.cshtml` | 2FA setup |
| `Manage/EnableAuthenticator.cshtml` | QR code for TOTP setup |

### When to Scaffold vs Write From Scratch

> [!example] Scaffold When:
> - You want a quick prototype with all auth flows working
> - Your UI requirements are close to the default -- just need styling changes
> - You want to override only a few specific pages (scaffold those pages only)

> [!example] Write From Scratch When:
> - You are building an API (no Razor Pages needed)
> - Your UI framework is React, Angular, or Blazor
> - You need significantly different UX flows (e.g., multi-step registration)
> - You want full control over every aspect of the auth experience

> [!summary] Section Summary
> `dotnet aspnet-codegenerator identity` scaffolds Razor Pages for all authentication flows. You can scaffold all pages or specific ones. Scaffolding is best for server-rendered apps with standard auth flows. For APIs or SPA frontends, write your own endpoints instead.

---

## Customizing Identity

### Password Options (Detailed)

```csharp
builder.Services.Configure<IdentityOptions>(options =>
{
    // Password complexity
    options.Password.RequireDigit = true;             // Require at least one 0-9
    options.Password.RequiredLength = 8;              // Minimum length
    options.Password.RequireNonAlphanumeric = true;   // Require !@#$%^&* etc.
    options.Password.RequireUppercase = true;          // Require A-Z
    options.Password.RequireLowercase = true;          // Require a-z
    options.Password.RequiredUniqueChars = 4;          // Minimum distinct characters
});
```

### Lockout Options

```csharp
options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
options.Lockout.MaxFailedAccessAttempts = 5;
options.Lockout.AllowedForNewUsers = true;  // Apply lockout to new accounts too
```

### Cookie Settings

```csharp
builder.Services.ConfigureApplicationCookie(options =>
{
    options.Cookie.Name = "MyApp.Auth";
    options.Cookie.HttpOnly = true;                    // Not accessible via JavaScript
    options.Cookie.SecurePolicy = CookieSecurePolicy.Always;  // HTTPS only
    options.Cookie.SameSite = SameSiteMode.Strict;
    options.ExpireTimeSpan = TimeSpan.FromHours(8);    // Cookie lifetime
    options.SlidingExpiration = true;                   // Refresh on activity
    options.LoginPath = "/Account/Login";               // Redirect path
    options.LogoutPath = "/Account/Logout";
    options.AccessDeniedPath = "/Account/AccessDenied";
});
```

### Token Lifespan

```csharp
builder.Services.Configure<DataProtectionTokenProviderOptions>(options =>
{
    options.TokenLifespan = TimeSpan.FromHours(3);  // Email/password reset tokens
});
```

> [!tip] Cookie HttpOnly and Secure
> Always set `HttpOnly = true` (prevents JavaScript access, mitigating XSS token theft) and `SecurePolicy = Always` (ensures cookies are only sent over HTTPS). These are security best practices, not optional settings.

> [!summary] Section Summary
> Identity is highly configurable. You can customize password complexity, lockout behavior, cookie properties (name, lifetime, security flags, paths), and token lifespans. Use `Configure<IdentityOptions>` for Identity settings and `ConfigureApplicationCookie` for cookie settings.

---

## Identity vs Manual Auth

| Aspect | ASP.NET Core Identity | Manual Authentication |
|--------|----------------------|----------------------|
| **Scope** | Full user management framework | Custom authentication logic only |
| **Setup effort** | Low -- add packages, configure, migrate | High -- build everything yourself |
| **Password hashing** | Built-in PBKDF2 with secure defaults | You must implement or choose a library |
| **2FA** | Built-in TOTP support | You must integrate a 2FA library |
| **Email confirmation** | Built-in token generation | You must implement token logic |
| **External logins** | Built-in OAuth/OIDC integration | You must handle OAuth flows manually |
| **Roles and claims** | Full management system | You must build your own |
| **Database** | Requires EF Core (by default) | Use any data access layer |
| **Flexibility** | Opinionated but extensible | Complete control |
| **Boilerplate** | More tables, more abstractions | Only what you need |
| **Best for** | Most web applications | Simple APIs, non-EF stores, microservices |

> [!tip] When to Choose Manual Auth
> If you are building a simple API with JWT authentication and only need to verify credentials against a single table, manual auth with `[Authorize]` and a custom JWT middleware might be simpler than bringing in Identity. However, the moment you need password resets, email confirmation, 2FA, or role management, Identity will save you significant development time.

> [!summary] Section Summary
> Identity is a comprehensive, opinionated framework that handles nearly every auth scenario out of the box. Manual auth gives you more control and fewer dependencies but requires building everything yourself. Choose Identity for most web apps; choose manual auth for simple APIs or when you need to avoid the EF Core dependency.

---

## AddIdentity vs AddDefaultIdentity vs AddIdentityCore

This is one of the most confusing aspects of Identity setup. ASP.NET Core offers three registration methods:

### AddIdentity<TUser, TRole>

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

### AddDefaultIdentity<TUser>

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

### AddIdentityCore<TUser>

```csharp
builder.Services.AddIdentityCore<ApplicationUser>(options => { ... })
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddDefaultTokenProviders();
```

- Registers the **minimal core** of Identity -- just `UserManager<T>`
- Does **not** add `SignInManager<T>` or `RoleManager<T>`
- Does **not** configure any authentication scheme
- **Use when**: Building APIs where you handle authentication separately (e.g., JWT) and just need user/password management

### Comparison Table

| Feature | `AddIdentity` | `AddDefaultIdentity` | `AddIdentityCore` |
|---------|--------------|---------------------|-------------------|
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

---

## Complete Real-World Example

This section walks through setting up Identity in an MVC application from scratch.

### Step 1: The Custom User Class

```csharp
// Models/ApplicationUser.cs
using Microsoft.AspNetCore.Identity;
using System.ComponentModel.DataAnnotations;

public class ApplicationUser : IdentityUser
{
    [Required]
    [MaxLength(100)]
    public string FullName { get; set; } = string.Empty;

    [MaxLength(50)]
    public string Department { get; set; } = string.Empty;

    public DateTime HireDate { get; set; }

    public bool IsActive { get; set; } = true;
}
```

### Step 2: The DbContext

```csharp
// Data/ApplicationDbContext.cs
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);

        // Optional: Additional configurations for ApplicationUser
        builder.Entity<ApplicationUser>(entity =>
        {
            entity.Property(u => u.FullName).HasMaxLength(100);
            entity.Property(u => u.Department).HasMaxLength(50);
        });
    }
}
```

### Step 3: Program.cs Setup

```csharp
// Program.cs
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Add EF Core
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));

// Add Identity
builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
    options.Password.RequireDigit = true;
    options.Password.RequiredLength = 8;
    options.Password.RequireNonAlphanumeric = true;
    options.Lockout.MaxFailedAccessAttempts = 5;
    options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
    options.User.RequireUniqueEmail = true;
    options.SignIn.RequireConfirmedEmail = false;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();

// Configure the authentication cookie
builder.Services.ConfigureApplicationCookie(options =>
{
    options.LoginPath = "/Account/Login";
    options.LogoutPath = "/Account/Logout";
    options.AccessDeniedPath = "/Account/AccessDenied";
    options.ExpireTimeSpan = TimeSpan.FromHours(8);
    options.SlidingExpiration = true;
    options.Cookie.HttpOnly = true;
});

builder.Services.AddControllersWithViews();

var app = builder.Build();

// Seed roles and admin user
using (var scope = app.Services.CreateScope())
{
    var services = scope.ServiceProvider;
    await SeedData.InitializeAsync(services);
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

### Step 4: Role Seeding

```csharp
// Data/SeedData.cs
using Microsoft.AspNetCore.Identity;

public static class SeedData
{
    public static async Task InitializeAsync(IServiceProvider serviceProvider)
    {
        var roleManager = serviceProvider
            .GetRequiredService<RoleManager<IdentityRole>>();
        var userManager = serviceProvider
            .GetRequiredService<UserManager<ApplicationUser>>();

        // Seed roles
        string[] roles = { "Admin", "Manager", "Employee" };
        foreach (var role in roles)
        {
            if (!await roleManager.RoleExistsAsync(role))
            {
                await roleManager.CreateAsync(new IdentityRole(role));
            }
        }

        // Seed admin user
        string adminEmail = "admin@company.com";
        if (await userManager.FindByEmailAsync(adminEmail) == null)
        {
            var admin = new ApplicationUser
            {
                UserName = adminEmail,
                Email = adminEmail,
                FullName = "System Administrator",
                Department = "IT",
                HireDate = new DateTime(2020, 1, 1),
                EmailConfirmed = true
            };

            var result = await userManager.CreateAsync(admin, "Admin@123456");
            if (result.Succeeded)
            {
                await userManager.AddToRoleAsync(admin, "Admin");
            }
        }
    }
}
```

### Step 5: View Models

```csharp
// Models/ViewModels/RegisterViewModel.cs
using System.ComponentModel.DataAnnotations;

public class RegisterViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;

    [Required]
    [StringLength(100, MinimumLength = 8)]
    [DataType(DataType.Password)]
    public string Password { get; set; } = string.Empty;

    [DataType(DataType.Password)]
    [Compare("Password", ErrorMessage = "Passwords do not match.")]
    public string ConfirmPassword { get; set; } = string.Empty;

    [Required]
    [MaxLength(100)]
    public string FullName { get; set; } = string.Empty;

    [MaxLength(50)]
    public string Department { get; set; } = string.Empty;
}

// Models/ViewModels/LoginViewModel.cs
public class LoginViewModel
{
    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;

    [Required]
    [DataType(DataType.Password)]
    public string Password { get; set; } = string.Empty;

    public bool RememberMe { get; set; }
}
```

### Step 6: AccountController

```csharp
// Controllers/AccountController.cs
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Mvc;

public class AccountController : Controller
{
    private readonly UserManager<ApplicationUser> _userManager;
    private readonly SignInManager<ApplicationUser> _signInManager;

    public AccountController(
        UserManager<ApplicationUser> userManager,
        SignInManager<ApplicationUser> signInManager)
    {
        _userManager = userManager;
        _signInManager = signInManager;
    }

    // GET: /Account/Register
    [HttpGet]
    public IActionResult Register() => View();

    // POST: /Account/Register
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Register(RegisterViewModel model)
    {
        if (!ModelState.IsValid)
            return View(model);

        var user = new ApplicationUser
        {
            UserName = model.Email,
            Email = model.Email,
            FullName = model.FullName,
            Department = model.Department,
            HireDate = DateTime.UtcNow
        };

        var result = await _userManager.CreateAsync(user, model.Password);

        if (result.Succeeded)
        {
            // Assign default role
            await _userManager.AddToRoleAsync(user, "Employee");

            // Sign in immediately after registration
            await _signInManager.SignInAsync(user, isPersistent: false);
            return RedirectToAction("Index", "Home");
        }

        // Add Identity errors to ModelState
        foreach (var error in result.Errors)
        {
            ModelState.AddModelError(string.Empty, error.Description);
        }

        return View(model);
    }

    // GET: /Account/Login
    [HttpGet]
    public IActionResult Login(string? returnUrl = null)
    {
        ViewData["ReturnUrl"] = returnUrl;
        return View();
    }

    // POST: /Account/Login
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Login(
        LoginViewModel model, string? returnUrl = null)
    {
        ViewData["ReturnUrl"] = returnUrl;

        if (!ModelState.IsValid)
            return View(model);

        var result = await _signInManager.PasswordSignInAsync(
            model.Email,
            model.Password,
            model.RememberMe,
            lockoutOnFailure: true);

        if (result.Succeeded)
        {
            return LocalRedirect(returnUrl ?? "/");
        }

        if (result.IsLockedOut)
        {
            return View("Lockout");
        }

        ModelState.AddModelError(string.Empty, "Invalid login attempt.");
        return View(model);
    }

    // POST: /Account/Logout
    [HttpPost]
    [ValidateAntiForgeryToken]
    public async Task<IActionResult> Logout()
    {
        await _signInManager.SignOutAsync();
        return RedirectToAction("Index", "Home");
    }

    // GET: /Account/AccessDenied
    [HttpGet]
    public IActionResult AccessDenied() => View();
}
```

> [!tip] Always Use LocalRedirect for Return URLs
> Use `LocalRedirect(returnUrl)` instead of `Redirect(returnUrl)` to prevent **open redirect attacks**. An attacker could craft a login URL with a `returnUrl` pointing to a malicious site. `LocalRedirect` ensures the redirect stays within your application.

> [!warning] Common Misconception
> `ValidateAntiForgeryToken` is essential on POST actions to prevent CSRF attacks. If you forget it, an attacker could forge a form submission from another site. Always include it on state-changing actions.

> [!summary] Section Summary
> A complete Identity setup involves: a custom `ApplicationUser` class, an `IdentityDbContext`, Identity service registration in `Program.cs`, role seeding, view models, and an `AccountController` with Register, Login, and Logout actions. Use `LocalRedirect` for return URLs and always include anti-forgery tokens.

---

## Comprehensive Summary

> [!tip] Complete Summary
> **ASP.NET Core Identity** is a full-featured user management framework that provides:
>
> **Core Components:**
> - `UserManager<T>` -- CRUD operations on users, password management, claims, tokens
> - `SignInManager<T>` -- sign-in/sign-out, external login flows, 2FA coordination
> - `RoleManager<T>` -- role creation and management
> - `IdentityDbContext<T>` -- EF Core integration with 7 pre-defined tables
>
> **Out-of-the-Box Features:**
> - User registration and authentication
> - PBKDF2 password hashing (600,000 iterations in .NET 8+)
> - Email confirmation and password reset via secure tokens
> - TOTP-based two-factor authentication
> - Account lockout after configurable failed attempts
> - Role-based and claims-based authorization support
> - External login provider integration (OAuth 2.0 / OpenID Connect)
>
> **Three Registration Methods:**
> - `AddIdentity<TUser, TRole>` -- full stack with roles, no default UI
> - `AddDefaultIdentity<TUser>` -- includes default UI, no roles unless `.AddRoles<T>()`
> - `AddIdentityCore<TUser>` -- minimal, for APIs (UserManager only)
>
> **Key Setup Points:**
> - `UseAuthentication()` must precede `UseAuthorization()` in the middleware pipeline
> - Always call `base.OnModelCreating()` in your DbContext
> - Always check `IdentityResult.Succeeded` after mutations
> - Use `LocalRedirect` for return URLs to prevent open redirect attacks
> - Seed roles and admin users at application startup
>
> Identity is the right choice for most ASP.NET Core web applications. For simple API scenarios or non-EF stores, consider manual authentication with `AddIdentityCore` or custom middleware.

---

## Related Topics

- [[Authentication Overview]] -- broader look at authentication in ASP.NET Core
- [[Cookie Authentication]] -- the authentication scheme Identity uses by default
- [[JWT Authentication]] -- token-based auth for APIs, often used with `AddIdentityCore`
- [[Authorization Policies]] -- defining fine-grained access rules using roles and claims
- [[Entity Framework Core]] -- the ORM that powers Identity's data storage
- [[Claims-Based Authorization]] -- using claims for attribute-based access control
- [[External Authentication Providers]] -- deep dive into Google, Microsoft, GitHub integration
- [[Data Protection API]] -- the underlying system Identity uses for token generation

---

## Further Reading

- [Microsoft Docs: Introduction to Identity on ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity)
- [Microsoft Docs: Scaffold Identity in ASP.NET Core projects](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/scaffold-identity)
- [Microsoft Docs: Configure ASP.NET Core Identity](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/identity-configuration)
- [Microsoft Docs: Account confirmation and password recovery](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/accconfirm)
- [Microsoft Docs: Multi-factor authentication in ASP.NET Core](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/mfa)
- [Andrew Lock: ASP.NET Core in Action (Manning)](https://www.manning.com/books/asp-net-core-in-action-third-edition) -- Chapters on Identity and Security
- [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
