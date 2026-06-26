---
tags: [csharp, ef-core, migrations, schema]
---

## Two Approaches to Schema Management

EF Core supports two workflows for how the database schema comes into existence:

| Approach          | Direction                              | Source of Truth |
| ----------------- | -------------------------------------- | --------------- |
| **Code First**    | C# classes → Migrations → Database     | Your code       |
| **Database First** | Existing Database → Scaffold → C# classes | The database    |

---

## Code First

You write entity classes first, then EF Core generates the database schema through [[Migrations Overview|migrations]].

```csharp
// 1. Write your entity
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}

// 2. Generate and apply migration
// dotnet ef migrations add AddProduct
// dotnet ef database update
// → EF creates the table for you
```

### When to Use

- **Greenfield projects** — starting from scratch with no existing database
- **You want code as the single source of truth** — schema changes are tracked in migration files alongside your code in version control
- **Domain-driven design** — model your domain objects first, let the database follow
- **Team environments** — migrations can be committed, reviewed, and merged like any other code

### Downsides

- Generated schema may not be optimal — tweak with [[Fluent API]] or [[Data Annotations]]
- Complex DB features (triggers, views, stored procedures) need manual SQL in migrations
- Migration merge conflicts in large teams

---

## Database First (Scaffolding)

The database already exists — EF Core reverse-engineers it into entity classes and a [[DbContext]].

```
dotnet ef dbcontext scaffold \
    "Server=localhost;Database=myapp;User=root;Password=pass" \
    Pomelo.EntityFrameworkCore.MySql \
    --output-dir Models
```

This generates:
- One entity class per table
- A `DbContext` with `DbSet<T>` properties and `OnModelCreating` configuration

### When to Use

- **Legacy / existing databases** — the DB is already built and in production
- **DBAs control the schema** — developers consume the schema, not own it
- **Multiple applications share the same database** — the DB is the contract
- **Database is the source of truth** — schema changes happen via SQL scripts, not migrations

### Downsides

- Re-scaffolding **overwrites** your customizations — use `partial` classes to keep custom logic separate
- No migration history — schema changes are managed outside EF Core
- Generated code can be verbose and needs cleanup

---

## Comparison

| Aspect              | Code First                             | Database First                         |
| ------------------- | -------------------------------------- | -------------------------------------- |
| **Source of truth**  | C# code                               | Database                               |
| **Schema changes**   | `dotnet ef migrations add`             | SQL scripts / DBA applies changes      |
| **Best for**         | New projects                           | Existing / legacy databases            |
| **Team workflow**    | Devs own schema                        | DBAs own schema                        |
| **EF Core command**  | `dotnet ef migrations add`             | `dotnet ef dbcontext scaffold`         |
| **Version control**  | Migration files tracked in git         | SQL scripts tracked separately         |
| **Flexibility**      | Full control via Fluent API            | Limited to what the DB gives you       |

---

## Hybrid Approach

Many teams start Database First to scaffold an existing DB, then switch to Code First migrations going forward:

1. Scaffold the existing database to generate entity classes
2. Create an initial migration that matches the current schema
3. From that point on, make all changes through Code First migrations

---

## Inspecting Generated SQL

Regardless of approach, you can always inspect the SQL that EF Core generates from your LINQ:

```csharp
var query = db.Products.Where(p => p.Price > 50);
Console.WriteLine(query.ToQueryString());
// Output: SELECT ... FROM Products WHERE Price > 50
```

This is useful for understanding what EF Core is doing under the hood and for performance tuning.

---

## See Also

- [[Migrations Overview]] — how Code First migrations work in detail
- [[DbContext]] — the central session class for both approaches
- [[EF Core Overview]] — EF Core fundamentals and architecture
- [[Seeding Data]] — populating initial data in both approaches
