---
tags:
  - csharp
  - ef-core
  - migrations
  - database
---

## What Are Migrations?

Migrations are **version control for your database schema**. Every time you change your entity classes (add a property, rename a table, change a relationship), the database needs to change too. Instead of writing raw SQL by hand, EF Core migrations compare your current model to the previous snapshot and generate C# code that, when executed, produces the exact DDL (CREATE, ALTER, DROP) to bring the database in sync.

Think of it like Git for your database structure: each migration is a commit that records what changed and how to undo it.

> [!ad-note] Why Not Just Recreate the Database?
> Dropping and recreating works during early development, but the moment you have real data (or a shared dev database), you need incremental changes. Migrations give you a repeatable, reviewable upgrade path from any schema version to any other.

---

## Two Tooling Options

You interact with migrations through one of two tools. They do the same thing -- pick whichever fits your workflow.

| Command | Package Manager Console (VS) | .NET CLI |
|---|---|---|
| Add a migration | `Add-Migration Name` | `dotnet ef migrations add Name` |
| Apply to database | `Update-Database` | `dotnet ef database update` |
| Remove last unapplied migration | `Remove-Migration` | `dotnet ef migrations remove` |
| List all migrations | `Get-Migration` | `dotnet ef migrations list` |
| Generate SQL script | `Script-Migration` | `dotnet ef migrations script` |
| Revert to a specific migration | `Update-Database PreviousMigrationName` | `dotnet ef database update PreviousMigrationName` |

> [!ad-note] CLI Prerequisite
> For the `dotnet ef` CLI, install the tool globally:
> ```bash
> dotnet tool install --global dotnet-ef
> ```
> Your project also needs the `Microsoft.EntityFrameworkCore.Design` NuGet package.

---

## The Core Workflow

The day-to-day cycle is simple:

1. **Change your entity classes** -- add a property, change a type, add a new entity, etc.
2. **Add a migration** -- EF Core diffs your model against the last snapshot and generates code.
3. **Review the generated code** -- always read what it plans to do before applying.
4. **Update the database** -- execute the migration against your database.

```text
Change Model  -->  Add-Migration  -->  Review Code  -->  Update-Database
     ^                                                         |
     |_________________________________________________________|
                       (repeat for next change)
```

---

## Migration File Structure

When you run `Add-Migration AddYearToCarTable`, EF Core creates a file like `20260609_AddYearToCarTable.cs` with two methods:

```csharp
public partial class AddYearToCarTable : Migration
{
    // What to do when applying this migration
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.AddColumn<int>(
            name: "Year",
            table: "Cars",
            type: "int",
            nullable: false,
            defaultValue: 0);   // SQL Server needs a default for existing rows
    }

    // What to do when reverting this migration
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropColumn(
            name: "Year",
            table: "Cars");
    }
}
```

- **`Up()`** -- the forward path. Contains the changes to apply.
- **`Down()`** -- the rollback path. Undoes exactly what `Up()` did.

EF Core also generates a **snapshot file** (`ModelSnapshot.cs`) that stores the full current state of your model. This is how it knows what changed next time you add a migration.

---

## Initial Migration

The very first migration creates all tables for your entire model:

```powershell
Add-Migration InitialCreate
Update-Database
```

The generated `Up()` will contain `CreateTable` calls for every entity in your [[DbContext]]. The `Down()` will drop them all.

---

## Example: Adding a New Column

Suppose your `Car` entity gains a `Color` property:

```csharp
public class Car
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Year { get; set; }
    public string Color { get; set; }   // <-- new property
}
```

```powershell
Add-Migration AddColorToCar
```

Generated migration:

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    migrationBuilder.AddColumn<string>(
        name: "Color",
        table: "Cars",
        type: "nvarchar(max)",
        nullable: true);    // string is nullable by default
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    migrationBuilder.DropColumn(
        name: "Color",
        table: "Cars");
}
```

Then apply:

```powershell
Update-Database
```

---

## Script Generation for Production

> [!ad-warning] Never Run `Update-Database` Against Production
> `Update-Database` requires a direct connection and runs with whatever permissions your app has. For production, generate a SQL script and hand it to your DBA or deployment pipeline.

Generate a script that covers all unapplied migrations:

```powershell
# PMC
Script-Migration

# CLI
dotnet ef migrations script
```

Generate a script between two specific migrations:

```powershell
# From migration A to migration B
dotnet ef migrations script AddYearToCarTable AddColorToCar
```

Generate an **idempotent** script (safe to run even if some migrations already applied):

```powershell
dotnet ef migrations script --idempotent
```

The idempotent flag wraps each migration in an `IF NOT EXISTS` check against the `__EFMigrationsHistory` table.

---

## Rolling Back Migrations

To revert to a previous state, specify the target migration name:

```powershell
# PMC -- rolls back everything after InitialCreate
Update-Database InitialCreate

# CLI
dotnet ef database update InitialCreate
```

To roll back **all** migrations (empty database):

```powershell
Update-Database 0
```

After rolling back, remove the now-unapplied migration files:

```powershell
Remove-Migration
```

> [!ad-note] `Remove-Migration` Only Works on the Last Unapplied Migration
> You can only remove migrations from the top of the stack. If you need to undo an older one, roll back to before it first, then remove in order.

---

## The __EFMigrationsHistory Table

EF Core tracks which migrations have been applied in a special table called `__EFMigrationsHistory`. It has two columns:

| Column | Purpose |
|---|---|
| `MigrationId` | The timestamped migration name (e.g., `20260609120000_InitialCreate`) |
| `ProductVersion` | The EF Core version that generated the migration |

When you run `Update-Database`, EF checks this table to determine which migrations are pending.

---

## Common Pitfalls

> [!ad-warning] Never Edit a Migration After It Has Been Applied to a Shared Database
> Once a migration has been run against a database that other developers or environments use, treat it as immutable. If you need to change something, create a **new** migration. Editing an applied migration causes the snapshot to diverge from reality, leading to broken subsequent migrations.

> [!ad-warning] Keep Migrations in Source Control
> Migration files and the snapshot file must be committed to your repository. Other developers need them to bring their local databases up to date. Never `.gitignore` migration files.

> [!ad-note] Merge Conflicts in Snapshot
> When two developers add migrations in parallel, the `ModelSnapshot.cs` file will conflict. Resolve the conflict, then run:
> ```powershell
> # Regenerate the snapshot from the merged migration history
> dotnet ef migrations remove   # if you have an empty/broken migration
> dotnet ef migrations add MergeFixup
> ```
> Alternatively, one developer can rebase and regenerate their migration on top of the other's.

---

## Quick Reference: Common Commands

```powershell
# --- Package Manager Console (Visual Studio) ---
Add-Migration MigrationName              # Generate migration code
Update-Database                          # Apply all pending migrations
Update-Database MigrationName            # Roll back/forward to specific migration
Remove-Migration                         # Delete last unapplied migration
Script-Migration                         # Generate SQL for all pending
Script-Migration -Idempotent             # Safe-to-rerun SQL script

# --- .NET CLI ---
dotnet ef migrations add MigrationName
dotnet ef database update
dotnet ef database update MigrationName
dotnet ef migrations remove
dotnet ef migrations list
dotnet ef migrations script
dotnet ef migrations script --idempotent
```

---

## Related Topics

- [[Fluent API Configuration]] -- configure the model that migrations translate into schema
- [[Seeding Data]] -- pre-populate data as part of migrations
- [[DbContext]] -- the central class that migrations are generated from
- [[Entity Classes]] -- the C# classes that map to database tables
