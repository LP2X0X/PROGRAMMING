---
tags: [csharp, ef-core, fluent-api, fundamentals]
---

## What Is the Fluent API

- The **Fluent API** is EF Core's most powerful and complete configuration mechanism.
- It uses a **method-chaining** syntax (hence "fluent") inside the `OnModelCreating` method of your [[DbContext]].
- Every call returns a builder object, so you chain configuration calls one after another without ever losing context.
- Think of it as giving EF Core precise instructions: "For this entity, this property should be required, have a max length of 100, and map to this column name."

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Car>()          // start configuring the Car entity
        .Property(c => c.Name)          // target the Name property
        .IsRequired()                   // NOT NULL
        .HasMaxLength(100);             // nvarchar(100)
}
```

- The entry point is always `modelBuilder.Entity<T>()`, which returns an `EntityTypeBuilder<T>`. From there, you branch into specific configuration areas.

---

## Why the Fluent API Exists

- While [[Entity Classes|conventions]] and Data Annotations cover common scenarios, many configurations are **only available through the Fluent API**. There is no attribute-based alternative for these.

### Configurations Exclusive to Fluent API

| Configuration | Why It Needs Fluent API |
|---|---|
| **Composite primary keys** | C# attributes don't support specifying column order reliably across all providers |
| **Global query filters** | A WHERE clause applied to every query — no attribute can express arbitrary LINQ predicates |
| **Many-to-many join table configuration** | Customizing the join table name, columns, or adding an explicit join entity |
| **Alternate keys** | Unique constraints that can serve as FK targets — a concept beyond what annotations support |
| **Filtered indexes** | Index with a SQL WHERE clause (e.g., unique only where not null) |
| **Inheritance mapping strategies** | TPH discriminator configuration, TPT table mapping, TPC strategy selection |
| **Table splitting / entity splitting** | Mapping multiple entities to one table, or one entity across multiple tables |
| **Shadow properties** | Properties that exist in the EF model and database but not in the C# class |
| **Computed columns with SQL expressions** | `HasComputedColumnSql(...)` — annotations can't hold arbitrary SQL |
| **Sequences** | Database sequences for value generation |
| **Model-level configuration** | Setting default schema, configuring value converters at scale |

```ad-note
title: The 80/20 Rule
Conventions handle ~80% of your mapping. Data Annotations handle another ~15% (simple constraints like `[Required]`, `[MaxLength]`). The Fluent API handles the remaining ~5% — but that 5% includes the most important architectural decisions: composite keys, query filters, inheritance strategies, and complex relationships.
```

---

## The Precedence Rule

- When multiple configuration mechanisms conflict, EF Core follows a strict precedence order:

```
Convention  <  Data Annotations  <  Fluent API
(lowest)                          (highest — always wins)
```

- If a convention says a string column is `nvarchar(max)`, but a `[MaxLength(200)]` annotation says 200, the annotation wins.
- If a `[MaxLength(200)]` annotation says 200 but the Fluent API says `.HasMaxLength(100)`, the **Fluent API wins**.
- This means the Fluent API is the **final authority**. If something isn't mapping the way you expect, check your Fluent API configuration first.

```ad-warning
title: Common Misconception
Some developers think Data Annotations override Fluent API because annotations are "closer to the code." The opposite is true. Fluent API **always** takes precedence. This catches people off guard when they add `[MaxLength(200)]` to a property but the Fluent API still enforces `HasMaxLength(100)`.
```

---

## Where You Write It

- All Fluent API configuration goes inside the `OnModelCreating` method override on your `DbContext`:

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Car> Cars { get; set; }
    public DbSet<Make> Makes { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure the Car entity
        modelBuilder.Entity<Car>(entity =>
        {
            entity.ToTable("Vehicles");             // table name
            entity.HasKey(c => c.Id);               // primary key
            
            entity.Property(c => c.Name)
                .IsRequired()                       // NOT NULL
                .HasMaxLength(100);                  // nvarchar(100)
            
            entity.HasOne(c => c.Make)              // relationship
                .WithMany(m => m.Cars)
                .HasForeignKey(c => c.MakeId);
        });

        // Configure the Make entity
        modelBuilder.Entity<Make>(entity =>
        {
            entity.Property(m => m.Name)
                .IsRequired()
                .HasMaxLength(50);
        });
    }
}
```

- Notice the **action delegate** syntax: `modelBuilder.Entity<Car>(entity => { ... })`. This groups all configuration for one entity inside a single lambda, keeping things organized.

---

## Entry Points and Builder Chain

- `modelBuilder.Entity<T>()` is always the starting point. It returns an `EntityTypeBuilder<T>`.
- From there, you branch into specific builders depending on what you're configuring:

```
modelBuilder.Entity<T>()
├── .Property(x => x.Prop)        → PropertyBuilder       → property config
├── .HasKey(x => x.Id)            → KeyBuilder             → primary key
├── .HasAlternateKey(x => x.Prop) → KeyBuilder             → alternate key (unique, FK-targetable)
├── .HasIndex(x => x.Prop)        → IndexBuilder           → index config
├── .HasOne(x => x.Nav)           → ReferenceNavigationBuilder → start of relationship chain
├── .HasMany(x => x.Nav)          → CollectionNavigationBuilder → start of relationship chain
├── .ToTable("Name")              → EntityTypeBuilder      → table mapping
├── .HasQueryFilter(x => ...)     → EntityTypeBuilder      → global query filter
├── .HasDiscriminator(...)        → DiscriminatorBuilder   → inheritance mapping
├── .Ignore(x => x.Prop)          → EntityTypeBuilder      → exclude property
└── .OwnsOne(x => x.Prop)         → OwnedNavigationBuilder → owned type config
```

- Each method returns an appropriate builder so you can keep chaining. This is the **fluent** in "Fluent API" — you flow from one configuration to the next.

```ad-tip
title: Read the IntelliSense
In Visual Studio or Rider, after typing `entity.`, IntelliSense will show you every available method. The method names are descriptive enough to discover most configurations without documentation. Methods starting with `Has` define something; methods starting with `Is` toggle a boolean; methods starting with `To` map to a database concept.
```

---

## The IEntityTypeConfiguration Pattern

- As your model grows, `OnModelCreating` can become enormous. The standard solution is to split configuration into **separate classes** — one per entity — using the `IEntityTypeConfiguration<T>` interface.

### Defining a Configuration Class

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

public class CarConfiguration : IEntityTypeConfiguration<Car>
{
    public void Configure(EntityTypeBuilder<Car> builder)
    {
        builder.ToTable("Vehicles");
        
        builder.HasKey(c => c.Id);
        
        builder.Property(c => c.Name)
            .IsRequired()
            .HasMaxLength(100);
        
        builder.Property(c => c.Price)
            .HasColumnType("decimal(10,2)");
        
        builder.HasOne(c => c.Make)
            .WithMany(m => m.Cars)
            .HasForeignKey(c => c.MakeId)
            .OnDelete(DeleteBehavior.Cascade);
        
        builder.HasQueryFilter(c => !c.IsDeleted);
    }
}
```

### Registering Configuration Classes

There are two ways to register these in `OnModelCreating`:

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Option 1: Register one by one
    modelBuilder.ApplyConfiguration(new CarConfiguration());
    modelBuilder.ApplyConfiguration(new MakeConfiguration());
    
    // Option 2: Scan the entire assembly (recommended)
    modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
}
```

```ad-note
title: ApplyConfigurationsFromAssembly Is the Standard Approach
`ApplyConfigurationsFromAssembly` scans the given assembly for all classes implementing `IEntityTypeConfiguration<T>` and applies them automatically. This means you never have to remember to register a new configuration class — just create it and it's discovered. This is the approach used in virtually all production EF Core projects.
```

### File Organization

- A common convention is to create a `Configurations` folder in your data project:

```
MyApp.Data/
├── AppDbContext.cs
├── Configurations/
│   ├── CarConfiguration.cs
│   ├── MakeConfiguration.cs
│   ├── FeatureConfiguration.cs
│   └── CarFeatureConfiguration.cs
├── Entities/
│   ├── Car.cs
│   ├── Make.cs
│   └── Feature.cs
└── Migrations/
```

```ad-tip
title: One Configuration Class Per Entity
Keep a strict 1:1 mapping between entity classes and configuration classes. Name the configuration `{EntityName}Configuration`. This makes it trivial to find where an entity's mapping is defined — search for `CarConfiguration` to find how `Car` maps to the database.
```

---

## Fluent API vs Data Annotations — When to Use Each

| Use Case | Recommended Approach | Reason |
|---|---|---|
| Simple constraints (`Required`, `MaxLength`) | Either — but Data Annotations also give you ASP.NET validation | Annotations are visible on the entity class |
| Table/column naming | Either | Personal preference |
| Composite keys | **Fluent API only** | Not possible with annotations |
| Global query filters | **Fluent API only** | Arbitrary LINQ predicates |
| Complex relationships | **Fluent API** | More explicit, easier to debug |
| Inheritance mapping | **Fluent API only** | Discriminator configuration, TPT/TPC setup |
| Index configuration | **Fluent API** (mostly) | Filtered indexes, include columns are Fluent-only |
| Shadow properties | **Fluent API only** | By definition, no C# property to attach an attribute to |
| When entity classes are in a separate library | **Fluent API** | Keeps EF Core dependencies out of the domain project |

```ad-tip
title: Clean Architecture Consideration
In Clean Architecture / DDD projects, entity (domain) classes should be free of infrastructure dependencies. Data Annotations require `System.ComponentModel.DataAnnotations` — an infrastructure concern. The Fluent API keeps all EF Core configuration in the data/infrastructure layer, leaving domain classes as pure POCOs with no EF Core references.
```

---

## See Also

- [[Property and Table Configuration]] — detailed property, table, shadow property, and query filter configuration
- [[Key and Index Configuration]] — primary keys, composite keys, alternate keys, and indexes
- [[Relationship Configuration]] — one-to-many, one-to-one, many-to-many, and delete behaviors
- [[Inheritance Mapping]] — TPH, TPT, and TPC inheritance strategies
- [[Entity Classes]] — the entity classes that the Fluent API configures
- [[Relationships]] — fundamentals of how EF Core models relationships
- [[DbContext]] — where `OnModelCreating` lives
- [[Fluent API Configuration]] — the earlier introduction in the Migrations and Schema section
