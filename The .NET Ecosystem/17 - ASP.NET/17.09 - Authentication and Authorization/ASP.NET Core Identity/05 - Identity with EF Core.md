---
tags:
  - csharp
  - asp-net-core
  - identity
  - authentication
  - security
---


## The DbContext

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

## Registering the DbContext

```csharp
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(
        builder.Configuration.GetConnectionString("DefaultConnection")));
```

## Running Migrations

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
