---
tags:
  - csharp
  - ef-core
  - seeding
  - migrations
---

## What Is Data Seeding?

**Data seeding** is pre-populating your database with initial data as part of your migration pipeline. Instead of manually inserting rows after deployment, you declare the data in C# and EF Core generates the INSERT statements inside a migration file.

This is ideal for data that **must exist** for the application to function: lookup tables, reference data, default configuration values, enum-like rows, or a default admin account.

---

## How It Works: HasData()

Seed data is declared inside `OnModelCreating` using `modelBuilder.Entity<T>().HasData()`:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Make>().HasData(
        new Make { Id = 1, Name = "Toyota" },
        new Make { Id = 2, Name = "Honda" },
        new Make { Id = 3, Name = "Ford" }
    );
}
```

When you run `Add-Migration SeedMakes`, EF Core generates:

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.InsertData(
        table: "Makes",
        columns: new[] { "Id", "Name" },
        values: new object[,]
        {
            { 1, "Toyota" },
            { 2, "Honda" },
            { 3, "Ford" }
        });
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DeleteData(
        table: "Makes",
        keyColumn: "Id",
        keyValues: new object[] { 1, 2, 3 });
}
```

---

## Critical Rules

> [!ad-warning] You MUST Provide Primary Key Values Explicitly
> Unlike normal inserts where the database auto-generates the `Id`, `HasData()` **requires you to specify the PK value**. EF needs the key to detect changes to seed data in future migrations.
> ```csharp
> // WRONG -- will throw at migration time
> new Make { Name = "Toyota" }
> 
> // CORRECT
> new Make { Id = 1, Name = "Toyota" }
> ```

> [!ad-warning] Navigation Properties Cannot Be Used
> You cannot set navigation properties in `HasData()`. Use the **foreign key property** instead:
> ```csharp
> // WRONG -- navigation property
> new Car { Id = 1, Name = "Camry", Make = someMakeObject }
> 
> // CORRECT -- foreign key value
> new Car { Id = 1, Name = "Camry", MakeId = 1 }
> ```

---

## Seeding Related Entities

Seed parent and child entities separately, linking them by FK values:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Seed parent table
    modelBuilder.Entity<Make>().HasData(
        new Make { Id = 1, Name = "Toyota" },
        new Make { Id = 2, Name = "Honda" }
    );

    // Seed child table (reference parent by FK)
    modelBuilder.Entity<Car>().HasData(
        new Car { Id = 1, Name = "Camry", Year = 2024, MakeId = 1 },
        new Car { Id = 2, Name = "Civic", Year = 2024, MakeId = 2 },
        new Car { Id = 3, Name = "Corolla", Year = 2023, MakeId = 1 }
    );
}
```

---

## How Seed Data Changes Are Tracked

EF Core compares the current `HasData()` declarations against the model snapshot from the previous migration. When you change seed data, the **next migration captures the diff**:

- **Add a new row** --> migration generates `InsertData`
- **Change a value** --> migration generates `UpdateData`
- **Remove a row** --> migration generates `DeleteData`

```csharp
// Original seeding
new Make { Id = 1, Name = "Toyota" }

// Changed to:
new Make { Id = 1, Name = "Toyota Motor Corporation" }

// Next migration generates:
migrationBuilder.UpdateData(
    table: "Makes",
    keyColumn: "Id",
    keyValue: 1,
    column: "Name",
    value: "Toyota Motor Corporation");
```

---

## When to Use HasData()

| Use Case | Good Fit? | Notes |
|---|---|---|
| Lookup/reference tables (countries, statuses) | Yes | Exactly what `HasData` was designed for |
| Enum-like reference data | Yes | Map enum values to rows |
| Default admin account | Maybe | Only if credentials are not sensitive |
| Test/demo data | No | Use a separate seeding mechanism |
| Large datasets (thousands of rows) | No | Use raw SQL or a post-deploy script |

---

## Alternative: Raw SQL in Migrations

For large datasets or complex seeding logic that `HasData()` cannot handle, inject raw SQL directly into a migration's `Up()` method:

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    // Create the table first (auto-generated)
    migrationBuilder.CreateTable(
        name: "ZipCodes",
        columns: table => new
        {
            Id = table.Column<int>(nullable: false)
                .Annotation("SqlServer:Identity", "1, 1"),
            Code = table.Column<string>(maxLength: 10),
            City = table.Column<string>(maxLength: 100)
        },
        constraints: table => table.PrimaryKey("PK_ZipCodes", x => x.Id));

    // Seed from a SQL script or inline SQL
    migrationBuilder.Sql(
        "INSERT INTO ZipCodes (Code, City) VALUES ('10001', 'New York')");

    // Or execute a script file
    // migrationBuilder.Sql(File.ReadAllText("Scripts/SeedZipCodes.sql"));
}
```

---

## Alternative: Custom Initialization Logic

For test/demo data or complex seeding that needs services (e.g., password hashing), create a separate seeding class and call it at startup:

```csharp
public static class DbInitializer
{
    public static void Seed(AppDbContext context)
    {
        // Only seed if the table is empty
        if (context.Makes.Any())
            return;

        var makes = new List<Make>
        {
            new Make { Name = "Toyota" },
            new Make { Name = "Honda" }
        };

        context.Makes.AddRange(makes);
        context.SaveChanges();   // auto-generated IDs are fine here
    }
}

// In Program.cs:
using (var scope = app.Services.CreateScope())
{
    var context = scope.ServiceProvider.GetRequiredService<AppDbContext>();
    context.Database.Migrate();     // apply pending migrations
    DbInitializer.Seed(context);    // seed data
}
```

> [!ad-note] HasData vs Custom Initialization
> `HasData()` embeds seed data **inside migrations** -- it's versioned, reviewable, and runs as part of schema updates. Custom initialization runs at **application startup** -- it's more flexible but not tracked by migrations. Use `HasData()` for stable reference data; use custom initialization for dynamic or environment-specific data.

---

## Organizing Seed Data with IEntityTypeConfiguration

Keep seeding alongside entity configuration using the `IEntityTypeConfiguration<T>` pattern from [[Fluent API Configuration]]:

```csharp
public class MakeConfiguration : IEntityTypeConfiguration<Make>
{
    public void Configure(EntityTypeBuilder<Make> builder)
    {
        builder.HasKey(m => m.Id);
        builder.Property(m => m.Name)
            .IsRequired()
            .HasMaxLength(50);

        // Seed data lives with the entity's configuration
        builder.HasData(
            new Make { Id = 1, Name = "Toyota" },
            new Make { Id = 2, Name = "Honda" },
            new Make { Id = 3, Name = "Ford" }
        );
    }
}
```

This keeps `OnModelCreating` clean and groups each entity's schema + seed data together.

---

## Related Topics

- [[Migrations Overview]] -- seed data changes generate new migrations
- [[Fluent API Configuration]] -- `HasData()` is part of the Fluent API
- [[Entity Classes]] -- the classes whose data you are seeding
- [[DbContext]] -- `OnModelCreating` is where `HasData()` is called
