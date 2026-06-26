---
tags:
  - csharp
  - ef-core
  - cli
  - migrations
  - database
---

# EF Core CLI Reference

> [!abstract] Overview
> A flag-by-flag reference for every `dotnet ef` command and its Package Manager Console (PMC) equivalent. This note is designed for quick lookup -- ctrl+F to the command you need, find the exact syntax, flags, and a practical example. For conceptual background on how migrations work, see [[Migrations Overview]].

---

## Table of Contents

- [Setup and Installation](#Setup%20and%20Installation)
- [Global Options](#Global%20Options)
- [dotnet ef migrations](#dotnet%20ef%20migrations)
- [dotnet ef database](#dotnet%20ef%20database)
- [dotnet ef dbcontext](#dotnet%20ef%20dbcontext)
- [PMC Quick Reference](#PMC%20Quick%20Reference)
- [Common Workflows](#Common%20Workflows)
- [Troubleshooting](#Troubleshooting)
- [Comprehensive Summary](#Comprehensive%20Summary)
- [Related Topics](#Related%20Topics)

---

## Setup and Installation

### Installing the CLI Tool

The `dotnet ef` commands are not part of the .NET SDK by default. You need to install the EF Core CLI as a global tool:

```bash
dotnet tool install --global dotnet-ef
```

To update to the latest version:

```bash
dotnet tool update --global dotnet-ef
```

### Required NuGet Package

Your **startup project** (or the project containing the [[DbContext]]) must reference the design-time package:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Design
```

This package is only needed at development time. It provides the build-time components that the CLI invokes to discover your `DbContext`, build the model, and generate migration code.

> [!ad-note] Runtime vs. Design-Time
> `Microsoft.EntityFrameworkCore.Design` is a development dependency. It does not ship with your published application. If you are using the PMC in Visual Studio, the equivalent package is `Microsoft.EntityFrameworkCore.Tools`.

### Verifying Installation

```bash
dotnet ef
```

If installed correctly, this prints the EF Core CLI version and the ASCII unicorn logo. If you see `"Could not execute because the specified command or file was not found"`, the tool is not installed or not on your PATH.

> [!summary] Section Summary
> - Install the CLI globally with `dotnet tool install --global dotnet-ef`.
> - The `Microsoft.EntityFrameworkCore.Design` NuGet package is required in your project.
> - Run `dotnet ef` with no arguments to verify the installation.
> - Use `dotnet tool update` to stay current with the latest EF Core tooling.

---

## Global Options

These flags can be appended to **any** `dotnet ef` command. They control which project to build, which `DbContext` to use, and output behavior.

| Flag | Short | Description |
|---|---|---|
| `--project` | `-p` | Path to the project containing the `DbContext` and migrations. Defaults to the current directory. |
| `--startup-project` | `-s` | Path to the startup project (the one with `Program.cs` / DI configuration). Used to resolve services and connection strings. |
| `--context` | `-c` | The fully qualified or simple name of the `DbContext` class to use. Required when the project contains multiple `DbContext` classes. |
| `--configuration` | | Build configuration to use (e.g., `Debug`, `Release`). Defaults to `Debug`. |
| `--framework` | | Target framework moniker when the project multi-targets (e.g., `net8.0`). |
| `--no-build` | | Skip building the project before running the command. Only use this if you have already built successfully. |
| `--verbose` | `-v` | Show detailed output including the SQL being executed and internal EF Core diagnostics. |
| `--no-color` | | Suppress ANSI color codes in output. Useful for CI logs. |
| `--prefix-output` | | Prefix each output line with its level (`info:`, `warn:`, `error:`). Useful for log parsing in CI/CD. |

> [!ad-note] --project vs --startup-project
> In a multi-project solution, `--project` points to the class library where your `DbContext` and migration files live. `--startup-project` points to the executable project (API, console app) that has the `appsettings.json` and DI container setup. EF Core builds the startup project, runs it briefly to get a configured `DbContext` instance, then uses the `--project` assembly for migration code generation.

> [!summary] Section Summary
> - `--project` / `-p` targets the DbContext project; `--startup-project` / `-s` targets the executable with configuration.
> - `--context` / `-c` disambiguates when multiple DbContexts exist.
> - `--no-build` skips compilation (use only after a clean build).
> - `--verbose` / `-v` is invaluable for debugging CLI issues.

---

## dotnet ef migrations

All subcommands for creating, removing, listing, scripting, and bundling migrations.

### migrations add

Creates a new migration by comparing the current model snapshot to the current entity classes.

```bash
dotnet ef migrations add <NAME> [options]
```

**PMC equivalent:** `Add-Migration <NAME>`

| Flag | Short | Description |
|---|---|---|
| `--output-dir` | `-o` | Directory for the generated migration files, relative to the project root. Defaults to `Migrations/`. |
| `--namespace` | `-n` | Namespace for the generated migration class. Defaults to the project's default namespace + `.Migrations`. |
| `--no-transactions` | | Do not wrap the migration in a transaction. Use for operations that cannot run inside a transaction (e.g., creating a columnstore index on SQL Server). |

**Example -- custom output directory:**

```bash
dotnet ef migrations add AddOrderTable \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api \
  --output-dir Data/Migrations
```

> [!ad-note] Naming Conventions
> Use descriptive PascalCase names that describe the change: `AddOrderStatusColumn`, `CreateAuditLogTable`, `RemoveObsoleteCustomerFields`. Avoid generic names like `Update1` -- when you have 50 migrations, you will want meaningful names.

---

### migrations remove

Deletes the most recent migration. If the migration has already been applied to the database, it also reverts it (runs the `Down()` method) unless `--force` is used.

```bash
dotnet ef migrations remove [options]
```

**PMC equivalent:** `Remove-Migration`

| Flag | Short | Description |
|---|---|---|
| `--force` | `-f` | Remove the migration files without checking whether the migration has been applied to the database. Does **not** revert the migration -- the database schema will be out of sync. |

**Example:**

```bash
dotnet ef migrations remove \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api
```

> [!ad-warning] Using --force Is Dangerous
> `--force` deletes the migration code files even if the migration has already been applied. Your database will still have the changes, but EF Core will no longer know about them. Use this only as a last resort when the database can be recreated.

---

### migrations list

Displays all migrations and their status (applied or pending).

```bash
dotnet ef migrations list [options]
```

**PMC equivalent:** `Get-Migration`

| Flag | Short | Description |
|---|---|---|
| `--connection` | | Connection string to use for checking applied status. Overrides the one from DI / `appsettings.json`. |
| `--no-connect` | | List migration files without connecting to the database. All migrations will show as status unknown. |

**Example:**

```bash
dotnet ef migrations list \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api
```

Output looks like:

```text
20260601120000_InitialCreate (Applied)
20260609143000_AddOrderTable (Applied)
20260615090000_AddOrderStatusColumn (Pending)
```

---

### migrations script

Generates a SQL script from migrations. This is the recommended way to deploy migrations to staging and production environments.

```bash
dotnet ef migrations script [FROM] [TO] [options]
```

**PMC equivalent:** `Script-Migration [FROM] [TO]`

| Flag | Short | Description |
|---|---|---|
| `--from` | | The starting migration (exclusive). Defaults to `0` (before the first migration). |
| `--to` | | The ending migration (inclusive). Defaults to the last migration. |
| `--idempotent` | `-i` | Generate an idempotent script that checks `__EFMigrationsHistory` before applying each migration. Safe to run multiple times. |
| `--no-transactions` | | Do not wrap each migration in a transaction in the generated script. |
| `--output` | `-o` | Output file path. If omitted, the script is written to stdout. |

**Example -- idempotent script to a file:**

```bash
dotnet ef migrations script --idempotent \
  --output ./scripts/migration-2026-06-15.sql \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api
```

**Example -- script between two specific migrations:**

```bash
dotnet ef migrations script AddOrderTable AddOrderStatusColumn \
  --output ./scripts/incremental.sql
```

> [!ad-note] Always Use --idempotent for Production Scripts
> Idempotent scripts are safe to run even if some migrations have already been applied. The script wraps each migration block in an existence check against `__EFMigrationsHistory`. This is the standard approach for CI/CD pipelines and DBA-reviewed deployments.

---

### migrations bundle

Creates a self-contained executable that applies migrations to a database. The bundle includes all pending migrations and can be deployed independently of the .NET SDK.

```bash
dotnet ef migrations bundle [options]
```

**PMC equivalent:** `Bundle-Migration` (EF Core 6.0+)

| Flag | Short | Description |
|---|---|---|
| `--output` | `-o` | Output file path for the bundle executable. Defaults to `efbundle` (or `efbundle.exe` on Windows). |
| `--force` | `-f` | Overwrite an existing bundle file. |
| `--self-contained` | | Bundle the .NET runtime so the target machine does not need .NET installed. |
| `--target-runtime` | `-r` | Target runtime identifier (e.g., `win-x64`, `linux-x64`). Required for cross-platform bundles. |

**Example -- self-contained bundle for Linux deployment:**

```bash
dotnet ef migrations bundle \
  --self-contained \
  --target-runtime linux-x64 \
  --output ./deploy/efbundle \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api
```

**Running the bundle:**

```bash
./efbundle --connection "Server=prod-db;Database=Inventory;User=deploy;Password=***"
```

> [!ad-note] Migration Bundles in CI/CD
> Migration bundles are the recommended deployment strategy starting with EF Core 6.0. They produce a single artifact that can be versioned, stored in a release pipeline, and executed on the target machine without needing the full SDK, source code, or project files. The bundle is an executable -- you just pass it a connection string.

---

### migrations has-pending-model-changes

Checks whether the current model has changes that have not been captured in a migration. Returns a non-zero exit code if there are pending model changes.

```bash
dotnet ef migrations has-pending-model-changes [options]
```

**PMC equivalent:** None (CLI only, EF Core 8.0+)

This command is useful in CI pipelines to enforce that developers always create a migration before merging:

```bash
dotnet ef migrations has-pending-model-changes \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api

# Exit code 0 = no pending changes, 1 = model has drifted from last migration
```

> [!summary] Section Summary
> - `migrations add` creates a new migration with optional `--output-dir` and `--namespace` control.
> - `migrations remove` deletes the last migration; `--force` skips the database revert check.
> - `migrations list` shows applied vs. pending status; `--no-connect` works without a database.
> - `migrations script` generates deployment SQL; always use `--idempotent` for production.
> - `migrations bundle` creates a standalone deployment executable (EF Core 6.0+).
> - `migrations has-pending-model-changes` is a CI guard for forgotten migrations (EF Core 8.0+).

---

## dotnet ef database

Commands that operate directly on the database.

### database update

Applies pending migrations to the database. Optionally targets a specific migration to roll forward or backward.

```bash
dotnet ef database update [MIGRATION] [options]
```

**PMC equivalent:** `Update-Database [MIGRATION]`

| Flag | Short | Description |
|---|---|---|
| `--connection` | | Connection string override. Takes precedence over the connection string resolved from DI / configuration. |

**Examples:**

```bash
# Apply all pending migrations
dotnet ef database update \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api

# Roll back to a specific migration (runs Down() for everything after it)
dotnet ef database update AddOrderTable

# Roll back ALL migrations (empty database, keep tables structure gone)
dotnet ef database update 0
```

> [!ad-warning] Rolling Back Is Destructive
> When you specify a migration name that is *before* the current state, EF Core runs the `Down()` methods of all migrations after that point. This drops columns, tables, and data. Always back up production data before rolling back.

---

### database drop

Drops the entire database. Asks for confirmation unless `--force` is supplied.

```bash
dotnet ef database drop [options]
```

**PMC equivalent:** `Drop-Database`

| Flag | Short | Description |
|---|---|---|
| `--force` | `-f` | Skip the confirmation prompt. |
| `--dry-run` | | Show which database would be dropped without actually dropping it. |

**Example:**

```bash
# See what would happen
dotnet ef database drop --dry-run \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api

# Actually drop (no confirmation prompt)
dotnet ef database drop --force \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api
```

> [!summary] Section Summary
> - `database update` applies migrations; target a specific migration name to roll forward/backward.
> - `database update 0` reverts all migrations.
> - `--connection` overrides the configured connection string.
> - `database drop` deletes the entire database; use `--dry-run` to preview and `--force` to skip confirmation.

---

## dotnet ef dbcontext

Commands for inspecting, scaffolding, and optimizing your [[DbContext]].

### dbcontext info

Displays information about the `DbContext` type: the provider, database name, and data source.

```bash
dotnet ef dbcontext info [options]
```

**PMC equivalent:** `Get-DbContext`

**Example:**

```bash
dotnet ef dbcontext info \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api
```

Output:

```text
Provider name: Microsoft.EntityFrameworkCore.SqlServer
Database name: InventoryDb
Data source: localhost
Options: None
```

---

### dbcontext list

Lists all `DbContext` types found in the project.

```bash
dotnet ef dbcontext list [options]
```

**PMC equivalent:** None (use `Get-DbContext` which shows info for the default or specified context)

---

### dbcontext scaffold

Reverse-engineers an existing database into entity classes and a `DbContext`. This is the [[Code First vs Database First|database-first]] approach.

```bash
dotnet ef dbcontext scaffold <CONNECTION> <PROVIDER> [options]
```

**PMC equivalent:** `Scaffold-DbContext <CONNECTION> <PROVIDER>`

| Flag | Short | Description |
|---|---|---|
| `--output-dir` | `-o` | Directory for generated entity classes. Defaults to the project root. |
| `--context-dir` | | Directory for the generated `DbContext` class. Defaults to `--output-dir`. |
| `--context` | `-c` | Name for the generated `DbContext` class. Defaults to the database name + `Context`. |
| `--namespace` | `-n` | Namespace for the generated entity classes. |
| `--context-namespace` | | Namespace for the generated `DbContext` class. Defaults to `--namespace`. |
| `--force` | `-f` | Overwrite existing files. Without this flag, the command fails if files already exist. |
| `--data-annotations` | `-d` | Use data annotations (attributes) instead of Fluent API for configuration where possible. |
| `--tables` | `-t` | Scaffold only specific tables. Can be specified multiple times. |
| `--schemas` | | Scaffold only tables from specific schemas. Can be specified multiple times. |
| `--no-onconfiguring` | | Suppress generation of the `OnConfiguring` method in the `DbContext`. Use this when you configure the context via DI. |
| `--no-pluralize` | | Do not pluralize `DbSet` property names. |
| `--use-database-names` | | Use exact table and column names from the database instead of C#-convention names. |

**Example -- scaffold selected tables with data annotations:**

```bash
dotnet ef dbcontext scaffold \
  "Server=localhost;Database=InventoryDb;Trusted_Connection=True;" \
  Microsoft.EntityFrameworkCore.SqlServer \
  --output-dir Models \
  --context-dir Data \
  --context InventoryContext \
  --data-annotations \
  --tables Orders --tables Products --tables Customers \
  --no-onconfiguring \
  --force
```

> [!ad-note] Re-scaffolding Overwrites Customizations
> `--force` overwrites all previously generated files. If you have customized the generated entities or `DbContext`, those changes are lost. Consider using partial classes for customizations, or switch to [[Code First vs Database First|code-first]] once the initial scaffold is done.

---

### dbcontext script

Generates the SQL script that creates the entire current model schema from scratch (no migrations involved).

```bash
dotnet ef dbcontext script [options]
```

**PMC equivalent:** `Script-DbContext`

This outputs the DDL for the full model as it exists in the current snapshot. It is useful for creating a fresh database without going through the migration history.

---

### dbcontext optimize

Pre-compiles the model to improve startup performance. Generates a compiled model that EF Core loads instead of building the model at runtime.

```bash
dotnet ef dbcontext optimize [options]
```

**PMC equivalent:** `Optimize-DbContext`

| Flag | Short | Description |
|---|---|---|
| `--output-dir` | `-o` | Directory for the generated compiled model files. Defaults to `CompiledModels/`. |
| `--namespace` | `-n` | Namespace for the generated compiled model classes. |

**Example:**

```bash
dotnet ef dbcontext optimize \
  --output-dir CompiledModels \
  --namespace Inventory.Data.CompiledModels \
  --project src/Inventory.Data \
  --startup-project src/Inventory.Api
```

After generating, register the compiled model in your `DbContext`:

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseModel(InventoryContextModel.Instance);
}
```

> [!ad-note] When to Use Compiled Models
> Compiled models primarily benefit applications with very large models (100+ entity types). For typical applications, the startup overhead of model building is negligible. Compiled models also have limitations -- they do not support some advanced features like global query filters defined with closures.

> [!summary] Section Summary
> - `dbcontext info` and `dbcontext list` are diagnostic commands for inspecting available contexts.
> - `dbcontext scaffold` reverse-engineers an existing database into C# code (database-first approach).
> - `dbcontext script` generates full-schema DDL without involving migrations.
> - `dbcontext optimize` pre-compiles the model for faster startup on large models.
> - Use `--no-onconfiguring` when configuring via DI; use `--force` with caution during re-scaffolding.

---

## PMC Quick Reference

Complete mapping between Package Manager Console commands and their .NET CLI equivalents.

| PMC Command | CLI Equivalent |
|---|---|
| `Add-Migration <Name>` | `dotnet ef migrations add <Name>` |
| `Remove-Migration` | `dotnet ef migrations remove` |
| `Update-Database` | `dotnet ef database update` |
| `Update-Database <Migration>` | `dotnet ef database update <Migration>` |
| `Get-Migration` | `dotnet ef migrations list` |
| `Script-Migration` | `dotnet ef migrations script` |
| `Script-Migration -Idempotent` | `dotnet ef migrations script --idempotent` |
| `Bundle-Migration` | `dotnet ef migrations bundle` |
| `Drop-Database` | `dotnet ef database drop` |
| `Scaffold-DbContext` | `dotnet ef dbcontext scaffold` |
| `Script-DbContext` | `dotnet ef dbcontext script` |
| `Optimize-DbContext` | `dotnet ef dbcontext optimize` |
| `Get-DbContext` | `dotnet ef dbcontext info` / `dbcontext list` |

> [!ad-note] PMC vs CLI: When to Use Which
> **PMC** is convenient when you are already in Visual Studio -- no need to type project paths since it uses the selected project in the Solution Explorer. **CLI** is the better choice for CI/CD pipelines, non-Visual Studio editors (VS Code, Rider), and scripting. Both invoke the same underlying EF Core design-time APIs.

> [!summary] Section Summary
> - Every PMC command has a CLI equivalent (the underlying APIs are identical).
> - PMC uses PowerShell-style flags (`-Idempotent`), CLI uses POSIX-style flags (`--idempotent`).
> - CLI is preferred for CI/CD and cross-platform workflows; PMC is convenient inside Visual Studio.

---

## Common Workflows

### Multi-Project Solutions

When your `DbContext` is in a class library and your configuration is in a separate web/console project:

```bash
dotnet ef migrations add AddAuditLog \
  --project src/MyApp.Data \
  --startup-project src/MyApp.Api
```

Every `dotnet ef` command will need both flags in this setup. To avoid repetition, run commands from the startup project directory and set `--project` only:

```bash
cd src/MyApp.Api
dotnet ef migrations add AddAuditLog --project ../MyApp.Data
```

---

### Multiple DbContexts

When a project contains more than one `DbContext`, you must specify which one to target:

```bash
dotnet ef migrations add AddTenantTable \
  --context TenantDbContext \
  --output-dir Migrations/Tenant \
  --project src/MyApp.Data \
  --startup-project src/MyApp.Api
```

Keep migrations for each context in separate directories using `--output-dir` to avoid confusion.

---

### CI/CD Deployment with Migration Bundles

A typical pipeline for deploying migrations as an artifact:

```bash
# Build step (CI agent)
dotnet ef migrations bundle \
  --self-contained \
  --target-runtime linux-x64 \
  --output ./artifacts/efbundle \
  --project src/MyApp.Data \
  --startup-project src/MyApp.Api

# Deploy step (target server)
./artifacts/efbundle \
  --connection "Server=prod-db;Database=MyApp;User=deploy;Password=${DB_PASSWORD}"
```

---

### Generating Idempotent Scripts for DBA Review

```bash
dotnet ef migrations script --idempotent \
  --output ./deploy/migration.sql \
  --project src/MyApp.Data \
  --startup-project src/MyApp.Api
```

Hand the generated `.sql` file to your DBA. The idempotent checks ensure it is safe to run even if parts of it have already been applied.

---

### Reverting a Migration

**If the migration has NOT been applied to the database:**

```bash
# Simply remove the migration files
dotnet ef migrations remove
```

**If the migration HAS been applied:**

```bash
# Step 1: Roll back the database to the previous migration
dotnet ef database update PreviousMigrationName

# Step 2: Remove the now-unapplied migration files
dotnet ef migrations remove
```

---

### Database-First Re-scaffold After Schema Changes

When the DBA modifies the production schema and you need to update your entities:

```bash
dotnet ef dbcontext scaffold \
  "Server=prod-db;Database=InventoryDb;User=readonly;Password=***" \
  Microsoft.EntityFrameworkCore.SqlServer \
  --output-dir Models \
  --context-dir Data \
  --context InventoryContext \
  --tables Orders --tables Products \
  --no-onconfiguring \
  --force
```

> [!ad-note] Partial Re-scaffold
> Use `--tables` to re-scaffold only the tables that changed. This limits the blast radius of `--force`, since only the specified entity files are overwritten.

> [!summary] Section Summary
> - Multi-project solutions require `--project` and `--startup-project` on every command.
> - Separate migration directories per `DbContext` with `--output-dir` and `--context`.
> - Migration bundles are the recommended CI/CD deployment mechanism.
> - Always revert an applied migration with `database update` before using `migrations remove`.
> - Use `--tables` during re-scaffold to limit which files are overwritten.

---

## Troubleshooting

### "No project was found"

```text
No project was found. Change the current directory or use the --project option.
```

**Cause:** The current directory does not contain a `.csproj` file, or the `--project` path is wrong.

**Fix:** Provide the correct path:

```bash
dotnet ef migrations list --project src/MyApp.Data --startup-project src/MyApp.Api
```

---

### "More than one DbContext was found"

```text
More than one DbContext was found. Specify which one to use. Use the '-c' option...
```

**Cause:** The project contains multiple classes that inherit from `DbContext`.

**Fix:** Specify the context explicitly:

```bash
dotnet ef migrations add InitTenant --context TenantDbContext
```

Use `dotnet ef dbcontext list` to see all available contexts.

---

### "Build failed"

```text
Build failed. Use dotnet build to see the errors.
```

**Cause:** The project has compilation errors. The CLI builds the project before running any command.

**Fix:** Run `dotnet build` separately to see the full error output, fix the errors, then retry. You can also use `--no-build` if you have already built successfully and want to skip the rebuild -- but be careful: if the build is stale, the CLI will operate on an outdated assembly.

---

### "The migration has already been applied to the database"

```text
The migration '20260615_AddOrderStatus' has already been applied to the database.
Revert it and try again. If the migration has been applied to other databases, consider reverting
its changes using a new migration instead.
```

**Cause:** You are trying to `migrations remove` a migration that has already been applied.

**Fix:** First revert the database, then remove:

```bash
dotnet ef database update PreviousMigrationName
dotnet ef migrations remove
```

---

### "Unable to create a 'DbContext' of type '...'"

**Cause:** EF Core cannot instantiate your `DbContext`. Common reasons:
- Missing parameterless constructor and no `IDesignTimeDbContextFactory<T>` implementation
- Connection string not configured in the startup project
- Missing DI registration for the `DbContext`

**Fix:** Implement `IDesignTimeDbContextFactory<T>` in the project containing your `DbContext`:

```csharp
public class InventoryContextFactory : IDesignTimeDbContextFactory<InventoryContext>
{
    public InventoryContext CreateDbContext(string[] args)
    {
        var optionsBuilder = new DbContextOptionsBuilder<InventoryContext>();
        optionsBuilder.UseSqlServer("Server=localhost;Database=InventoryDb;Trusted_Connection=True;");
        return new InventoryContext(optionsBuilder.Options);
    }
}
```

> [!ad-note] IDesignTimeDbContextFactory
> This factory is only used by the EF Core tooling at design time. It has no effect on your application at runtime. It is the most reliable way to ensure the CLI can always instantiate your `DbContext`, regardless of how your application configures DI.

---

### "Your target project doesn't match your migrations assembly"

**Cause:** The `--project` flag points to a different assembly than the one configured in `MigrationsAssembly()` in your `DbContext` options.

**Fix:** Ensure consistency. If your `DbContext` configuration specifies:

```csharp
options.UseSqlServer(connectionString, x => x.MigrationsAssembly("MyApp.Data"));
```

Then `--project` must point to the `MyApp.Data` project.

> [!summary] Section Summary
> - Most CLI errors stem from incorrect `--project`, `--startup-project`, or `--context` flags.
> - Always run `dotnet build` first to isolate compilation errors from tooling errors.
> - Implement `IDesignTimeDbContextFactory<T>` as a fallback for design-time context creation.
> - When removing an applied migration, revert the database first with `database update`.
> - Ensure `MigrationsAssembly()` configuration matches the `--project` target.

---

## Comprehensive Summary

> [!tip] Complete Summary
> The EF Core CLI (`dotnet ef`) is a global .NET tool that provides three command groups: **migrations** (add, remove, list, script, bundle, has-pending-model-changes), **database** (update, drop), and **dbcontext** (info, list, scaffold, script, optimize). Every command accepts global options for targeting the correct project (`--project`), startup project (`--startup-project`), and DbContext (`--context`).
>
> For **migration management**, `migrations add` generates migration code with optional `--output-dir` and `--namespace` control. `migrations remove` deletes the last migration (revert first if applied). `migrations list` shows applied vs. pending status. For **deployment**, `migrations script --idempotent` generates DBA-safe SQL scripts, while `migrations bundle` creates standalone executables for CI/CD pipelines (EF Core 6.0+). `migrations has-pending-model-changes` guards CI builds against forgotten migrations (EF Core 8.0+).
>
> For **database operations**, `database update` applies migrations (specify a migration name to roll forward or backward, or `0` to revert everything). `database drop` removes the entire database.
>
> For **reverse engineering**, `dbcontext scaffold` generates entity classes and a DbContext from an existing database, with fine-grained control over output directories, namespaces, table selection, and naming conventions. `dbcontext optimize` pre-compiles the model for startup performance on large models.
>
> Every CLI command has a PMC equivalent for Visual Studio users. The underlying design-time APIs are identical -- choose the tool that fits your workflow. For production deployments, always prefer idempotent scripts or migration bundles over direct `database update`.

---

## Related Topics

- [[Migrations Overview]] -- conceptual guide to how EF Core migrations work
- [[Code First vs Database First]] -- choosing your development approach
- [[DbContext]] -- the central class that all CLI commands operate on
- [[Seeding Data]] -- populating initial data through migrations
- [[Fluent API Configuration]] -- configuring the model that drives migration generation
- [[Entity Classes]] -- the C# classes that map to database tables
