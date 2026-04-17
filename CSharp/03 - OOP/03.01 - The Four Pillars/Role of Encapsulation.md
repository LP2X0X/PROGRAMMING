---
tags:
 - csharp
 - oop
 - encapsulation
---

Encapsulation is the first pillar of OOP. It has two complementary aspects: **hiding implementation details** and **protecting object state**.

## Hiding Implementation Details

Encapsulation lets a class expose a simple public surface while keeping its internal complexity hidden. The caller only needs to know *what* an object does, not *how* it does it.

```csharp
DatabaseReader dbReader = new DatabaseReader();
dbReader.Open(@"C:\AutoLot.mdf");
// work with data...
dbReader.Close();
```

Behind `Open()` and `Close()`, there could be dozens of steps — locating the file, acquiring a handle, parsing headers, managing buffers, releasing resources. None of that leaks out to the caller. This separation means:

- **Internal changes don't break callers.** The class can swap its file-reading strategy, add caching, or change its error-recovery logic without touching any code that uses it.
- **Reduced cognitive load.** The caller thinks in terms of meaningful operations, not low-level mechanics.

## Protecting Object State (Data Hiding)

The second aspect is controlling access to an object's data. Fields should be `private`, `protected`, or `internal` — never `public`. The outside world interacts with state through controlled gateways.

```csharp
public class Employee
{
    private string _name;
    private decimal _salary;

    public string Name
    {
        get => _name;
        set => _name = !string.IsNullOrWhiteSpace(value)
            ? value
            : throw new ArgumentException("Name cannot be empty.");
    }

    public decimal Salary
    {
        get => _salary;
        set => _salary = value >= 0
            ? value
            : throw new ArgumentOutOfRangeException("Salary cannot be negative.");
    }
}
```

If `_salary` were public, any code could set it to `-50000` with no checks. By routing access through a property, the object enforces its own rules.

### Gateways for Controlled Access

| Mechanism      | Use Case                                                        |
| -------------- | --------------------------------------------------------------- |
| **Properties** | Validate, transform, or lazily compute field values             | 
| **Methods**    | Perform operations that involve multiple fields or side effects |
| **Indexers**   | Provide collection-like access with bounds checking             |

## Access Modifiers at a Glance

| Modifier             | Visible To                                                  |
| -------------------- | ----------------------------------------------------------- |
| `private`            | Only the containing class                                   |
| `protected`          | The containing class and its subclasses                     | 
| `internal`           | Any code in the same assembly                               |
| `protected internal` | Same assembly **or** subclasses (union)                     |
| `private protected`  | Subclasses **within** the same assembly only (intersection) |

Choosing the right modifier is part of encapsulation design — expose the minimum necessary surface.

## Why It Matters

- **Invariant enforcement.** The object guarantees its own consistency. A `BankAccount` can ensure the balance never goes negative without relying on callers to check first.
- **Loose coupling.** Code that depends on a public interface is insulated from internal restructuring. You can refactor freely behind the boundary.
- **Easier debugging.** When state can only change through controlled paths, you know exactly where to look when something goes wrong — set a breakpoint in the property setter, not across every file that touches a public field.
- **Testability.** A well-encapsulated class can be tested through its public API without needing to reach into private internals.


## Common Pitfalls

- **Anemic encapsulation** — making every field a public auto-property (`{ get; set; }`) with no validation. The syntax looks encapsulated, but there's no actual protection.
- **Leaking mutable references** — returning a private `List<T>` directly lets callers modify internal state. Return `IReadOnlyList<T>` or a copy instead.
- **Over-exposure** — marking members `public` "just in case" they're needed. Start with `private` and widen access only when there's a concrete reason.
