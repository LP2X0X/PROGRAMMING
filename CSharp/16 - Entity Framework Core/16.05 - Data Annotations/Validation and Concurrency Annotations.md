---
tags: [csharp, ef-core, data-annotations, validation, concurrency]
---

## Overview

- This note covers annotations that perform **data validation** (ensuring correct values) and **concurrency control** (detecting conflicting updates).
- Validation annotations do double duty: they configure the database schema **and** power ASP.NET model validation.
- Concurrency annotations are EF Core-specific — they have no effect in ASP.NET validation.
- All annotations in this note come from `System.ComponentModel.DataAnnotations`.

```csharp
using System.ComponentModel.DataAnnotations;
```

---

## `[Required]` — NOT NULL + Validation

- **EF Core effect**: the column is `NOT NULL` in the database.
- **ASP.NET effect**: the field must have a non-null, non-empty value (for strings) to pass `ModelState.IsValid`.

### Basic Usage

```csharp
public class Product
{
    public int Id { get; set; }

    [Required]
    [MaxLength(200)]
    public string Name { get; set; }  // NOT NULL nvarchar(200)
}
```

### With Custom Error Message

```csharp
[Required(ErrorMessage = "Product name is required")]
[MaxLength(200)]
public string Name { get; set; }
```

- The `ErrorMessage` is used by ASP.NET validation (shown in forms, returned in API error responses).
- EF Core ignores `ErrorMessage` — it only cares about the nullability.

### `[Required]` and Value Types

```csharp
public class Order
{
    public int Id { get; set; }

    [Required]
    public decimal Total { get; set; }    // decimal is already NOT NULL by convention (value type)

    [Required]
    public DateTime OrderDate { get; set; } // same — value types are NOT NULL by convention
}
```

```ad-note
`[Required]` on a value type (`int`, `decimal`, `DateTime`, `bool`, etc.) is **redundant for EF Core** — value types are already `NOT NULL` by convention. However, it still triggers ASP.NET validation (prevents the default `0` / `DateTime.MinValue` from being submitted without explicit user input in some binding scenarios).
```

### `[Required]` and Nullable Reference Types (NRT)

- In .NET 6+ projects, **nullable reference types** are enabled by default.
- A non-nullable `string` property (no `?`) is already treated as `NOT NULL` by EF Core.
- Adding `[Required]` is redundant for EF Core mapping but adds ASP.NET validation behavior (prevents empty strings).

```csharp
// With NRT enabled:
public string Name { get; set; }     // already NOT NULL in EF Core (NRT convention)

[Required]
public string Name { get; set; }     // still NOT NULL, but also validated by ASP.NET

public string? Name { get; set; }    // nullable in both EF Core and ASP.NET
```

```ad-tip
**Practical guidance with NRT enabled**: Use non-nullable types for required properties and nullable types (`?`) for optional ones. Add `[Required]` only when you also want ASP.NET validation (e.g., the entity doubles as a form model).
```

### `[Required]` on Navigation Properties

- Placing `[Required]` on a reference navigation makes the **relationship** required (FK becomes NOT NULL).
- See [[Key and Relationship Annotations]] for detailed coverage.

```csharp
[Required]
public Customer Customer { get; set; }  // CustomerId will be NOT NULL
```

### `AllowEmptyStrings` Property

```csharp
[Required(AllowEmptyStrings = true)]
public string Notes { get; set; }
// NOT NULL in DB, but ASP.NET allows "" (empty string) — only blocks null
```

- By default, `[Required]` blocks both `null` and `""` in ASP.NET validation.
- `AllowEmptyStrings = true` permits empty strings but still blocks `null`.

---

## `[ConcurrencyCheck]` — Optimistic Concurrency on Specific Columns

- **Optimistic concurrency** means: "Don't lock the row. Instead, check at save time whether anyone else changed it since you loaded it."
- `[ConcurrencyCheck]` tells EF Core to include this column in the `WHERE` clause of `UPDATE` and `DELETE` statements.

### How It Works

1. You load an entity. EF Core remembers the **original value** of the `[ConcurrencyCheck]` property.
2. You modify the entity and call `SaveChanges()`.
3. EF Core generates: `UPDATE ... SET ... WHERE Id = @Id AND CheckedColumn = @OriginalValue`
4. If another user changed `CheckedColumn` between step 1 and step 3, the `WHERE` matches **0 rows**.
5. EF Core detects 0 rows affected and throws `DbUpdateConcurrencyException`.

### Example

```csharp
public class BankAccount
{
    public int Id { get; set; }
    public string AccountHolder { get; set; }

    [ConcurrencyCheck]
    public decimal Balance { get; set; }
}
```

#### Scenario: Two Users Withdraw Simultaneously

```
User A loads account: Balance = 1000
User B loads account: Balance = 1000

User A sets Balance = 800 (withdraw 200), saves:
  UPDATE BankAccounts SET Balance = 800 WHERE Id = 1 AND Balance = 1000
  → 1 row affected ✓ — succeeds

User B sets Balance = 700 (withdraw 300), saves:
  UPDATE BankAccounts SET Balance = 700 WHERE Id = 1 AND Balance = 1000
  → 0 rows affected ✗ — Balance is now 800, not 1000
  → DbUpdateConcurrencyException thrown!
```

### Generated SQL

```sql
-- Without [ConcurrencyCheck]:
UPDATE BankAccounts SET Balance = @NewBalance WHERE Id = @Id

-- With [ConcurrencyCheck]:
UPDATE BankAccounts SET Balance = @NewBalance WHERE Id = @Id AND Balance = @OriginalBalance
```

### Multiple ConcurrencyCheck Columns

- You can apply `[ConcurrencyCheck]` to **multiple** properties. All of them will appear in the `WHERE` clause.

```csharp
public class Document
{
    public int Id { get; set; }

    [ConcurrencyCheck]
    public string Title { get; set; }

    [ConcurrencyCheck]
    public string LastModifiedBy { get; set; }

    public string Content { get; set; }
}
// WHERE clause: ... WHERE Id = @Id AND Title = @OrigTitle AND LastModifiedBy = @OrigModifiedBy
```

```ad-warning
title: Choosing the Right Columns
Be selective about which columns you mark with `[ConcurrencyCheck]`. If you check columns that change frequently (e.g., `LastAccessTime`), you'll get false conflicts. Ideal candidates are columns that represent the "business state" you care about protecting.
```

---

## `[Timestamp]` — Row Version Concurrency Token

- `[Timestamp]` maps to a **database-managed row version** that automatically changes every time the row is modified.
- On SQL Server, this maps to the `rowversion` data type (8-byte binary, auto-incremented by the database engine).
- The property must be `byte[]`.

### Basic Usage

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }

    [Timestamp]
    public byte[] RowVersion { get; set; }  // maps to rowversion in SQL Server
}
```

### How It Works

1. You load a `Product`. EF Core stores the current `RowVersion` value (e.g., `0x00000000000007D1`).
2. You change `Price` and call `SaveChanges()`.
3. EF Core generates:
   ```sql
   UPDATE Products
   SET Price = @NewPrice
   WHERE Id = @Id AND RowVersion = 0x00000000000007D1
   ```
4. The database updates the row **and automatically increments** `RowVersion` to a new value.
5. If another user modified the row between step 1 and step 3, the `RowVersion` won't match, 0 rows are updated, and EF Core throws `DbUpdateConcurrencyException`.

### Resulting SQL (SQL Server)

```sql
CREATE TABLE Products (
    Id         int             IDENTITY(1,1) PRIMARY KEY,
    Name       nvarchar(max)   NULL,
    Price      decimal(18,2)   NOT NULL,
    RowVersion rowversion      NOT NULL        -- auto-managed by SQL Server
);
```

```ad-note
title: [Timestamp] vs [ConcurrencyCheck] — When to Use Which

| Aspect | `[Timestamp]` | `[ConcurrencyCheck]` |
| --- | --- | --- |
| **Value management** | Database auto-generates and auto-increments | You manage the value in application code |
| **Detects** | Any change to the row (any column) | Changes only to the checked column(s) |
| **Property type** | Must be `byte[]` | Any type |
| **Provider support** | Best on SQL Server (`rowversion`); other providers may differ | Provider-agnostic |
| **Columns in WHERE** | 1 (`RowVersion`) | N (each checked column) |
| **Reliability** | Very high — guaranteed to change on every update | Depends on what you check |

**Recommendation**: Use `[Timestamp]` when you're on SQL Server and want to detect **any** concurrent modification. Use `[ConcurrencyCheck]` when you need provider-agnostic concurrency or want to check only specific columns.
```

### `[Timestamp]` on Non-SQL Server Databases

```ad-warning
title: Provider Differences
- **SQL Server**: `[Timestamp]` maps to `rowversion` — fully automatic, highly reliable.
- **PostgreSQL (Npgsql)**: Use `[ConcurrencyCheck]` on an `xmin` shadow property, or use Fluent API with `.IsRowVersion()`. The `[Timestamp]` attribute may not work as expected.
- **SQLite**: No native row version. You'd typically use a manually managed integer column with `[ConcurrencyCheck]`.
- **MySQL/MariaDB**: No native row version. Use a `DATETIME` column with `ON UPDATE CURRENT_TIMESTAMP` and `[ConcurrencyCheck]`.

If you're targeting multiple providers, `[ConcurrencyCheck]` is the safer choice.
```

---

## Handling `DbUpdateConcurrencyException`

- When a concurrency conflict occurs, EF Core throws `DbUpdateConcurrencyException`.
- You must catch it and decide how to resolve the conflict.

### Three Resolution Strategies

| Strategy | Description | When to Use |
| --- | --- | --- |
| **Client wins** (force overwrite) | Overwrite the database with the current values | When the latest save should always win |
| **Database wins** (discard changes) | Reload from database, discard local changes | When the first save should win |
| **Merge** | Compare original, current, and database values; merge intelligently | When you want to preserve non-conflicting changes |

### Full Exception Handling Pattern

```csharp
public async Task UpdateProductPrice(int productId, decimal newPrice)
{
    using var db = new AppDbContext();
    var product = await db.Products.FindAsync(productId);

    product.Price = newPrice;

    bool saved = false;
    while (!saved)
    {
        try
        {
            await db.SaveChangesAsync();
            saved = true;
        }
        catch (DbUpdateConcurrencyException ex)
        {
            var entry = ex.Entries.Single();

            // Get the current values in the database
            var dbValues = await entry.GetDatabaseValuesAsync();

            if (dbValues == null)
            {
                // The row was deleted by another user
                throw new InvalidOperationException(
                    "The product was deleted by another user.");
            }

            // STRATEGY 1: Client wins — force overwrite
            // Update the RowVersion to the current DB value, then retry
            entry.OriginalValues.SetValues(dbValues);
            // Loop will retry SaveChangesAsync with the updated RowVersion

            // STRATEGY 2: Database wins — discard local changes
            // entry.CurrentValues.SetValues(dbValues);
            // entry.OriginalValues.SetValues(dbValues);
            // saved = true;  // nothing to save, we accepted the DB state

            // STRATEGY 3: Merge — compare and decide per property
            // var proposedValues = entry.CurrentValues;
            // var originalValues = entry.OriginalValues;
            // foreach (var prop in proposedValues.Properties)
            // {
            //     var proposedVal = proposedValues[prop];
            //     var originalVal = originalValues[prop];
            //     var dbVal = dbValues[prop];
            //     // Custom logic: pick proposedVal or dbVal for each property
            //     proposedValues[prop] = /* your merge logic */;
            // }
            // entry.OriginalValues.SetValues(dbValues);
        }
    }
}
```

### Simplified "Client Wins" Pattern

```csharp
try
{
    await db.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{
    // Reload the entity from the database (gets fresh RowVersion)
    foreach (var entry in ex.Entries)
    {
        await entry.ReloadAsync();
    }
    // Re-apply your changes
    product.Price = newPrice;
    await db.SaveChangesAsync();  // retries with the current RowVersion
}
```

```ad-tip
**In web applications**, the most user-friendly approach is often to show the conflict to the user:
1. Load the entity, display it in a form, store the `RowVersion` in a hidden field.
2. When the user submits, set the `RowVersion` back on the entity before saving.
3. If `DbUpdateConcurrencyException` occurs, show both versions to the user and let them choose.

This is better than silently overwriting or discarding — the user makes the decision.
```

---

## Dual-Purpose Annotations — EF Core vs ASP.NET Validation

- Some annotations affect the database schema. Some trigger validation. Some do both.
- This distinction matters because developers often assume all annotations affect EF Core.

| Annotation | EF Core Effect | ASP.NET Validation Effect |
| --- | --- | --- |
| `[Required]` | `NOT NULL` column | Required field validation |
| `[MaxLength(n)]` | `nvarchar(n)` column | **None** |
| `[StringLength(n)]` | `nvarchar(n)` column | Max length validation |
| `[StringLength(n, MinimumLength = m)]` | `nvarchar(n)` column | Min + max length validation |
| `[Range(min, max)]` | **None** (EF ignores) | Range validation |
| `[EmailAddress]` | **None** | Email format validation |
| `[Phone]` | **None** | Phone format validation |
| `[Url]` | **None** | URL format validation |
| `[CreditCard]` | **None** | Credit card format validation |
| `[RegularExpression(pattern)]` | **None** | Regex validation |
| `[Compare("OtherProp")]` | **None** | Equality comparison validation |
| `[ConcurrencyCheck]` | Included in WHERE clause | **None** |
| `[Timestamp]` | `rowversion` column | **None** |

```ad-warning
title: Annotations EF Core Silently Ignores
`[Range]`, `[EmailAddress]`, `[Phone]`, `[Url]`, `[CreditCard]`, `[RegularExpression]`, and `[Compare]` are **validation-only** annotations. EF Core reads them during model building but does **nothing** with them — no CHECK constraints, no column type changes, nothing.

If you need database-level enforcement of these rules, use:
- Fluent API `HasCheckConstraint()` for CHECK constraints
- Raw SQL in a migration for complex constraints
- Database triggers (last resort)
```

### Example: Entity with Both EF and Validation Annotations

```csharp
public class Customer
{
    public int Id { get; set; }

    [Required(ErrorMessage = "Name is required")]                   // EF: NOT NULL | Validation: required
    [StringLength(100, MinimumLength = 2,
        ErrorMessage = "Name must be 2-100 characters")]            // EF: nvarchar(100) | Validation: length check
    public string Name { get; set; }

    [Required]
    [EmailAddress(ErrorMessage = "Invalid email")]                  // EF: nothing | Validation: format check
    [MaxLength(256)]                                                // EF: nvarchar(256) | Validation: nothing
    public string Email { get; set; }

    [Range(18, 150, ErrorMessage = "Age must be 18-150")]           // EF: nothing | Validation: range check
    public int Age { get; set; }

    [Phone]                                                         // EF: nothing | Validation: phone format
    [MaxLength(20)]                                                 // EF: nvarchar(20)
    public string PhoneNumber { get; set; }

    [Timestamp]                                                     // EF: rowversion | Validation: nothing
    public byte[] RowVersion { get; set; }
}
```

**What EF Core creates in the database:**

```sql
CREATE TABLE Customers (
    Id          int           IDENTITY(1,1) PRIMARY KEY,
    Name        nvarchar(100) NOT NULL,          -- from [Required] + [StringLength(100)]
    Email       nvarchar(256) NOT NULL,          -- from [Required] + [MaxLength(256)]
    Age         int           NOT NULL,          -- value type convention (NOT [Range])
    PhoneNumber nvarchar(20)  NULL,              -- from [MaxLength(20)]
    RowVersion  rowversion    NOT NULL           -- from [Timestamp]
);
-- Note: NO check constraints for email format, age range, or phone format
```

---

## Custom Validation Attributes

- You can create custom validation attributes by inheriting from `ValidationAttribute`.
- These work with ASP.NET validation but EF Core ignores them (just like `[Range]`).

```csharp
public class FutureDateAttribute : ValidationAttribute
{
    protected override ValidationResult IsValid(object value, ValidationContext context)
    {
        if (value is DateTime date && date <= DateTime.Now)
        {
            return new ValidationResult(ErrorMessage ?? "Date must be in the future");
        }
        return ValidationResult.Success;
    }
}

// Usage:
public class Event
{
    public int Id { get; set; }

    [Required]
    public string Title { get; set; }

    [FutureDate(ErrorMessage = "Event date must be in the future")]
    public DateTime EventDate { get; set; }
    // EF Core: datetime2 NOT NULL (ignores [FutureDate])
    // ASP.NET: validates that the date is in the future
}
```

---

## Concurrency in Practice — Putting It All Together

### Typical Entity with Full Concurrency Support

```csharp
public class Invoice
{
    public int Id { get; set; }

    [Required]
    [MaxLength(50)]
    public string InvoiceNumber { get; set; }

    [Required]
    public int CustomerId { get; set; }
    public Customer Customer { get; set; }

    [Column(TypeName = "decimal(12,2)")]
    public decimal TotalAmount { get; set; }

    [Required]
    [MaxLength(20)]
    public string Status { get; set; }           // Draft, Sent, Paid, Cancelled

    [ConcurrencyCheck]                            // detect if status was changed by another user
    public DateTime LastModified { get; set; }

    [Timestamp]                                   // detect ANY row modification
    public byte[] RowVersion { get; set; }
}
```

```ad-note
Using **both** `[ConcurrencyCheck]` and `[Timestamp]` on the same entity is valid but usually unnecessary. `[Timestamp]` already detects every modification. The `[ConcurrencyCheck]` on `LastModified` here is redundant because `RowVersion` catches everything. In practice, pick one strategy:
- `[Timestamp]` alone for comprehensive protection (SQL Server).
- `[ConcurrencyCheck]` on specific columns for targeted protection or cross-provider support.
```

### Controller / Service Layer Pattern

```csharp
// ASP.NET Controller using [Timestamp] for concurrency
[HttpPut("{id}")]
public async Task<IActionResult> UpdateInvoice(int id, InvoiceUpdateDto dto)
{
    var invoice = await _db.Invoices.FindAsync(id);
    if (invoice == null) return NotFound();

    // Set values from DTO
    invoice.TotalAmount = dto.TotalAmount;
    invoice.Status = dto.Status;
    invoice.LastModified = DateTime.UtcNow;

    // Set the RowVersion from the client (sent in the request)
    // This is critical — without it, concurrency detection won't work
    _db.Entry(invoice).Property(i => i.RowVersion).OriginalValue = dto.RowVersion;

    try
    {
        await _db.SaveChangesAsync();
        return Ok(invoice);
    }
    catch (DbUpdateConcurrencyException)
    {
        return Conflict(new
        {
            Message = "This invoice was modified by another user. Please reload and try again.",
            CurrentVersion = (await _db.Invoices.AsNoTracking()
                .FirstOrDefaultAsync(i => i.Id == id))
        });
    }
}
```

```ad-tip
**Always send `RowVersion` to the client and back**. In a web API, include it in the response DTO. When the client sends an update, include the `RowVersion` it received. Set `OriginalValue` to that version before saving. This is the "hidden field" pattern adapted for APIs.
```

---

## Cross-References

- [[Data Annotations Overview]] — full annotation reference, dual-purpose table, precedence rules
- [[Table and Column Annotations]] — `[MaxLength]`, `[StringLength]`, `[Column]`, `[NotMapped]`
- [[Key and Relationship Annotations]] — `[Key]`, `[Required]` on navigations, `[ForeignKey]`
- [[Change Tracking]] — how EF Core tracks entity state (related to concurrency detection)
- [[CRUD Operations]] — `SaveChanges` and exception handling patterns
- [[Fluent API Overview]] — Fluent equivalents: `.IsRequired()`, `.IsConcurrencyToken()`, `.IsRowVersion()`
