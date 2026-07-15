---
tags:
  - solid
  - isp
  - interface-segregation
---

## 🔹 Definition

The **Interface Segregation Principle (ISP)** is the fourth principle in [[SOLID Overview|SOLID]], stated by Robert C. Martin:

> **"Clients should not be forced to depend on interfaces they do not use."**

The core idea: **many small, specific interfaces are better than one large, general-purpose interface**. When an interface grows too large (a "fat interface"), it forces implementing classes to provide method bodies for operations they have no business performing. This leads to dummy implementations, `NotSupportedException` throws, and brittle code that breaks when unrelated parts of the interface change.

A fat interface couples unrelated concerns together. If `ClassA` only needs methods 1-3 and `ClassB` only needs methods 4-6, but both depend on an interface exposing all six, then changes to methods 4-6 ripple into `ClassA` even though it never calls them. ISP says: split that interface so each client depends only on the slice it actually uses.

---

## 🔹 Why It Matters

**1. Prevents [[3 - Liskov Substitution Principle|LSP]] violations.** When a class is forced to implement methods it cannot meaningfully support, the only options are throwing `NotSupportedException` or writing empty no-op bodies. Both violate [[3 - Liskov Substitution Principle|LSP]] because callers expecting those methods to work get surprising failures at runtime. ==Every `throw new NotSupportedException()` inside an interface implementation is both an ISP violation (the interface is too broad) and an LSP violation (the implementation cannot honor the contract).==

**2. Reduces recompilation and redeployment scope.** In compiled languages like C#, changing an interface forces recompilation of every assembly that references it. If a fat interface changes a method that only one implementer uses, every other implementer and every consumer must still be recompiled. With segregated interfaces, changes are localized -- modifying `IScanner` does not force recompilation of classes that only implement `IPrinter`.

**3. Simplifies testing and mocking.** When unit testing a class that depends on a 10-method interface but only calls 2, you must still set up mocks or stubs for all 10 methods. Segregated interfaces mean your test doubles only need to satisfy the small contract the class actually uses. Less mock setup means clearer, faster, more focused tests.

**4. Decreases coupling.** Depending on a fat interface means depending on every method signature, return type, and parameter type in that interface. A change to any of those (even methods you never call) can break your compilation. Narrow interfaces keep the dependency surface minimal.

**5. Improves readability and discoverability.** When a class constructor takes `IWorker` with 12 methods, it is unclear which methods actually matter to that class. When it takes `IWorkable` and `IReportWriter`, the dependency intent is self-documenting. Reviewers immediately understand what capabilities the class needs.

**6. Enables independent deployment.** In microservice or modular architectures, small interfaces can be deployed as separate assemblies or NuGet packages. Consumers only reference the interface assembly they need, reducing their dependency footprint.

---

## 🔹 Violation Example: Fat IWorker Interface

Consider a workplace system that models different kinds of workers. The naive approach is a single interface covering everything a worker might do:

```csharp
public interface IWorker
{
    void Work();
    void Eat();
    void Sleep();
    void AttendMeeting();
    void WriteReport();
    void SubmitTimesheet();
}
```

Now implement `FullTimeEmployee` -- this works fine because a full-time employee genuinely does all six things:

```csharp
public class FullTimeEmployee : IWorker
{
    public string Name { get; }

    public FullTimeEmployee(string name)
    {
        Name = name;
    }

    public void Work()
    {
        Console.WriteLine($"{Name} is working on assigned tasks.");
    }

    public void Eat()
    {
        Console.WriteLine($"{Name} is eating lunch in the break room.");
    }

    public void Sleep()
    {
        Console.WriteLine($"{Name} is resting during the night shift break.");
    }

    public void AttendMeeting()
    {
        Console.WriteLine($"{Name} is attending the weekly standup.");
    }

    public void WriteReport()
    {
        Console.WriteLine($"{Name} is writing the project status report.");
    }

    public void SubmitTimesheet()
    {
        Console.WriteLine($"{Name} submitted the weekly timesheet.");
    }
}
```

Now add a `Robot` -- robots do not eat, sleep, or attend meetings, but the fat interface forces implementations:

```csharp
public class Robot : IWorker
{
    public string Model { get; }

    public Robot(string model)
    {
        Model = model;
    }

    public void Work()
    {
        Console.WriteLine($"Robot {Model} is performing assembly line operations.");
    }

    // Robot cannot eat -- forced to implement anyway
    public void Eat()
    {
        throw new NotSupportedException("Robots do not eat.");
    }

    // Robot cannot sleep -- forced to implement anyway
    public void Sleep()
    {
        throw new NotSupportedException("Robots do not sleep.");
    }

    // Robot cannot attend meetings -- forced to implement anyway
    public void AttendMeeting()
    {
        throw new NotSupportedException("Robots do not attend meetings.");
    }

    public void WriteReport()
    {
        Console.WriteLine($"Robot {Model} generated an automated production report.");
    }

    public void SubmitTimesheet()
    {
        Console.WriteLine($"Robot {Model} logged operational hours automatically.");
    }
}
```

And a `Contractor` -- contractors do not attend internal meetings, but the fat interface forces that too:

```csharp
public class Contractor : IWorker
{
    public string Name { get; }

    public Contractor(string name)
    {
        Name = name;
    }

    public void Work()
    {
        Console.WriteLine($"Contractor {Name} is working on the contracted deliverable.");
    }

    public void Eat()
    {
        Console.WriteLine($"Contractor {Name} is eating lunch.");
    }

    public void Sleep()
    {
        Console.WriteLine($"Contractor {Name} is resting off-site.");
    }

    // Contractors are external -- they don't attend internal meetings
    public void AttendMeeting()
    {
        throw new NotSupportedException("Contractors do not attend internal meetings.");
    }

    public void WriteReport()
    {
        Console.WriteLine($"Contractor {Name} submitted the deliverable report.");
    }

    public void SubmitTimesheet()
    {
        Console.WriteLine($"Contractor {Name} submitted the invoice timesheet.");
    }
}
```

**What goes wrong:**

- Any code that iterates over `IEnumerable<IWorker>` and calls `Eat()` will crash at runtime when it hits the `Robot`.
- A method like `ScheduleLunchBreak(IWorker worker)` compiles just fine for `Robot`, giving zero compile-time safety -- the error only surfaces at runtime.
- Every `throw new NotSupportedException()` is a textbook [[3 - Liskov Substitution Principle|LSP]] violation: you cannot substitute a `Robot` anywhere an `IWorker` is expected without risking exceptions.
- If someone adds a seventh method to `IWorker` (say `TakeVacation()`), every implementer -- including `Robot` -- must be updated, even if the change is irrelevant to them.

---

## 🔹 Fixed Example: Segregated Interfaces

Split the fat interface by role -- each interface represents one cohesive capability:

```csharp
public interface IWorkable
{
    void Work();
}

public interface IFeedable
{
    void Eat();
    void Sleep();
}

public interface IMeetingAttendee
{
    void AttendMeeting();
}

public interface IReportWriter
{
    void WriteReport();
}

public interface ITimesheetSubmitter
{
    void SubmitTimesheet();
}
```

Now each class implements only the interfaces that apply to it:

```csharp
public class FullTimeEmployee : IWorkable, IFeedable, IMeetingAttendee,
                                 IReportWriter, ITimesheetSubmitter
{
    public string Name { get; }

    public FullTimeEmployee(string name)
    {
        Name = name;
    }

    public void Work()
    {
        Console.WriteLine($"{Name} is working on assigned tasks.");
    }

    public void Eat()
    {
        Console.WriteLine($"{Name} is eating lunch in the break room.");
    }

    public void Sleep()
    {
        Console.WriteLine($"{Name} is resting during the night shift break.");
    }

    public void AttendMeeting()
    {
        Console.WriteLine($"{Name} is attending the weekly standup.");
    }

    public void WriteReport()
    {
        Console.WriteLine($"{Name} is writing the project status report.");
    }

    public void SubmitTimesheet()
    {
        Console.WriteLine($"{Name} submitted the weekly timesheet.");
    }
}
```

`Robot` only implements the capabilities it genuinely has -- no more throwing:

```csharp
public class Robot : IWorkable, IReportWriter, ITimesheetSubmitter
{
    public string Model { get; }

    public Robot(string model)
    {
        Model = model;
    }

    public void Work()
    {
        Console.WriteLine($"Robot {Model} is performing assembly line operations.");
    }

    public void WriteReport()
    {
        Console.WriteLine($"Robot {Model} generated an automated production report.");
    }

    public void SubmitTimesheet()
    {
        Console.WriteLine($"Robot {Model} logged operational hours automatically.");
    }
}
```

`Contractor` implements all applicable interfaces except `IMeetingAttendee`:

```csharp
public class Contractor : IWorkable, IFeedable, IReportWriter, ITimesheetSubmitter
{
    public string Name { get; }

    public Contractor(string name)
    {
        Name = name;
    }

    public void Work()
    {
        Console.WriteLine($"Contractor {Name} is working on the contracted deliverable.");
    }

    public void Eat()
    {
        Console.WriteLine($"Contractor {Name} is eating lunch.");
    }

    public void Sleep()
    {
        Console.WriteLine($"Contractor {Name} is resting off-site.");
    }

    public void WriteReport()
    {
        Console.WriteLine($"Contractor {Name} submitted the deliverable report.");
    }

    public void SubmitTimesheet()
    {
        Console.WriteLine($"Contractor {Name} submitted the invoice timesheet.");
    }
}
```

**What improved:**

- `Robot` no longer has any methods it cannot support. No `NotSupportedException`, no [[3 - Liskov Substitution Principle|LSP]] violation.
- A method `ScheduleLunchBreak(IFeedable worker)` will not even compile if you pass a `Robot`. The type system catches the error at compile time.
- Adding a new method to `IMeetingAttendee` only affects `FullTimeEmployee` -- `Robot` and `Contractor` are untouched.
- Tests for a class that depends on `IWorkable` only need to mock `Work()`, not six methods.

---

## 🔹 Real-World Example 1: Printer Interface

**The violation -- a fat `IPrinter` interface:**

```csharp
public interface IPrinter
{
    void Print(string document);
    void Scan(string document);
    void Fax(string document, string faxNumber);
    void Staple(string document);
}

public class AllInOnePrinter : IPrinter
{
    public void Print(string document)
    {
        Console.WriteLine($"Printing: {document}");
    }

    public void Scan(string document)
    {
        Console.WriteLine($"Scanning: {document}");
    }

    public void Fax(string document, string faxNumber)
    {
        Console.WriteLine($"Faxing '{document}' to {faxNumber}");
    }

    public void Staple(string document)
    {
        Console.WriteLine($"Stapling: {document}");
    }
}

public class BasicInkjetPrinter : IPrinter
{
    public void Print(string document)
    {
        Console.WriteLine($"Inkjet printing: {document}");
    }

    // Basic inkjet has no scanner
    public void Scan(string document)
    {
        throw new NotSupportedException("This printer does not have a scanner.");
    }

    // Basic inkjet has no fax modem
    public void Fax(string document, string faxNumber)
    {
        throw new NotSupportedException("This printer does not support faxing.");
    }

    // Basic inkjet has no stapler
    public void Staple(string document)
    {
        throw new NotSupportedException("This printer does not have a stapler.");
    }
}
```

A `BasicInkjetPrinter` only prints, yet it must carry three dead methods that throw. Any code accepting `IPrinter` and calling `Scan()` will crash at runtime if it gets a `BasicInkjetPrinter`.

**The fix -- segregated interfaces:**

```csharp
public interface IPrinter
{
    void Print(string document);
}

public interface IScanner
{
    void Scan(string document);
}

public interface IFax
{
    void Fax(string document, string faxNumber);
}

public interface IStapler
{
    void Staple(string document);
}

// Convenience composition for devices that support everything
public interface IMultiFunctionDevice : IPrinter, IScanner, IFax, IStapler
{
}

public class AllInOnePrinter : IMultiFunctionDevice
{
    public void Print(string document)
    {
        Console.WriteLine($"Printing: {document}");
    }

    public void Scan(string document)
    {
        Console.WriteLine($"Scanning: {document}");
    }

    public void Fax(string document, string faxNumber)
    {
        Console.WriteLine($"Faxing '{document}' to {faxNumber}");
    }

    public void Staple(string document)
    {
        Console.WriteLine($"Stapling: {document}");
    }
}

public class BasicInkjetPrinter : IPrinter
{
    public void Print(string document)
    {
        Console.WriteLine($"Inkjet printing: {document}");
    }
}

public class ScannerOnlyDevice : IScanner
{
    public void Scan(string document)
    {
        Console.WriteLine($"Scanning: {document}");
    }
}
```

`BasicInkjetPrinter` now implements only `IPrinter`. There are no dead methods, no exceptions, and no way to accidentally call `Scan()` on it -- the compiler prevents it. Code that needs scanning asks for `IScanner`, making the required capabilities explicit.

Consumer code becomes self-documenting:

```csharp
public class DocumentArchiver
{
    private readonly IScanner _scanner;
    private readonly IPrinter _printer;

    // Constructor clearly communicates: this class needs scanning and printing
    public DocumentArchiver(IScanner scanner, IPrinter printer)
    {
        _scanner = scanner;
        _printer = printer;
    }

    public void ArchiveAndPrint(string document)
    {
        _scanner.Scan(document);
        _printer.Print(document);
    }
}

public class FaxService
{
    private readonly IFax _fax;

    // Only needs fax capability -- doesn't care about printing or scanning
    public FaxService(IFax fax) => _fax = fax;

    public void SendFax(string document, string number)
    {
        _fax.Fax(document, number);
    }
}
```

---

## 🔹 Real-World Example 2: Repository Pattern

**The violation -- a full CRUD repository interface for every entity:**

```csharp
public interface IRepository<T> where T : class
{
    T? GetById(int id);
    IEnumerable<T> GetAll();
    IEnumerable<T> Find(Expression<Func<T, bool>> predicate);
    void Add(T entity);
    void AddRange(IEnumerable<T> entities);
    void Update(T entity);
    void Delete(T entity);
    void DeleteRange(IEnumerable<T> entities);
}
```

This works fine for entities that support full CRUD:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
}

public class ProductRepository : IRepository<Product>
{
    private readonly List<Product> _products = new();

    public Product? GetById(int id) => _products.FirstOrDefault(p => p.Id == id);
    public IEnumerable<Product> GetAll() => _products.AsReadOnly();
    public IEnumerable<Product> Find(Expression<Func<Product, bool>> predicate)
        => _products.AsQueryable().Where(predicate).ToList();
    public void Add(Product entity) => _products.Add(entity);
    public void AddRange(IEnumerable<Product> entities) => _products.AddRange(entities);
    public void Update(Product entity)
    {
        var existing = GetById(entity.Id);
        if (existing is not null)
        {
            existing.Name = entity.Name;
            existing.Price = entity.Price;
        }
    }
    public void Delete(Product entity) => _products.Remove(entity);
    public void DeleteRange(IEnumerable<Product> entities)
    {
        foreach (var entity in entities)
            _products.Remove(entity);
    }
}
```

But some entities must NEVER be modified or deleted. Audit logs are a classic case -- regulatory requirements demand that audit records are immutable:

```csharp
public class AuditLogEntry
{
    public int Id { get; set; }
    public DateTime Timestamp { get; set; }
    public string Action { get; set; } = string.Empty;
    public string UserId { get; set; } = string.Empty;
    public string Details { get; set; } = string.Empty;
}

// Problem: audit logs must NEVER be modified or deleted, but the interface forces it
public class AuditLogRepository : IRepository<AuditLogEntry>
{
    private readonly List<AuditLogEntry> _logs = new();

    public AuditLogEntry? GetById(int id) => _logs.FirstOrDefault(l => l.Id == id);
    public IEnumerable<AuditLogEntry> GetAll() => _logs.AsReadOnly();
    public IEnumerable<AuditLogEntry> Find(Expression<Func<AuditLogEntry, bool>> predicate)
        => _logs.AsQueryable().Where(predicate).ToList();

    public void Add(AuditLogEntry entity) => _logs.Add(entity);
    public void AddRange(IEnumerable<AuditLogEntry> entities) => _logs.AddRange(entities);

    // Audit logs are immutable records -- updates must not be allowed
    public void Update(AuditLogEntry entity)
    {
        throw new NotSupportedException("Audit logs cannot be modified.");
    }

    // Audit logs must never be deleted -- regulatory requirement
    public void Delete(AuditLogEntry entity)
    {
        throw new NotSupportedException("Audit logs cannot be deleted.");
    }

    public void DeleteRange(IEnumerable<AuditLogEntry> entities)
    {
        throw new NotSupportedException("Audit logs cannot be deleted.");
    }
}
```

The `AuditLogRepository` is forced to expose `Update`, `Delete`, and `DeleteRange` which it must actively prevent. This is dangerous: a developer working with `IRepository<AuditLogEntry>` sees those methods in IntelliSense and might call them, only to get a runtime exception.

**The fix -- segregated repository interfaces:**

```csharp
public interface IReadRepository<T> where T : class
{
    T? GetById(int id);
    IEnumerable<T> GetAll();
    IEnumerable<T> Find(Expression<Func<T, bool>> predicate);
}

public interface IWriteRepository<T> where T : class
{
    void Add(T entity);
    void AddRange(IEnumerable<T> entities);
    void Update(T entity);
}

public interface IDeleteRepository<T> where T : class
{
    void Delete(T entity);
    void DeleteRange(IEnumerable<T> entities);
}

// Compose the full CRUD interface from the smaller ones
public interface IRepository<T> : IReadRepository<T>, IWriteRepository<T>, IDeleteRepository<T>
    where T : class
{
}
```

Products support full CRUD -- implement the composed interface:

```csharp
public class ProductRepository : IRepository<Product>
{
    private readonly List<Product> _products = new();

    public Product? GetById(int id) => _products.FirstOrDefault(p => p.Id == id);
    public IEnumerable<Product> GetAll() => _products.AsReadOnly();
    public IEnumerable<Product> Find(Expression<Func<Product, bool>> predicate)
        => _products.AsQueryable().Where(predicate).ToList();
    public void Add(Product entity) => _products.Add(entity);
    public void AddRange(IEnumerable<Product> entities) => _products.AddRange(entities);
    public void Update(Product entity)
    {
        var existing = GetById(entity.Id);
        if (existing is not null)
        {
            existing.Name = entity.Name;
            existing.Price = entity.Price;
        }
    }
    public void Delete(Product entity) => _products.Remove(entity);
    public void DeleteRange(IEnumerable<Product> entities)
    {
        foreach (var entity in entities)
            _products.Remove(entity);
    }
}
```

Audit logs are append-only: read + add, but no update or delete:

```csharp
public class AuditLogRepository : IReadRepository<AuditLogEntry>, IWriteRepository<AuditLogEntry>
{
    private readonly List<AuditLogEntry> _logs = new();

    public AuditLogEntry? GetById(int id) => _logs.FirstOrDefault(l => l.Id == id);
    public IEnumerable<AuditLogEntry> GetAll() => _logs.AsReadOnly();
    public IEnumerable<AuditLogEntry> Find(Expression<Func<AuditLogEntry, bool>> predicate)
        => _logs.AsQueryable().Where(predicate).ToList();
    public void Add(AuditLogEntry entity) => _logs.Add(entity);
    public void AddRange(IEnumerable<AuditLogEntry> entities) => _logs.AddRange(entities);

    // Note: IWriteRepository includes Update. If truly immutable, split further:
    public void Update(AuditLogEntry entity)
    {
        throw new NotSupportedException("Audit logs cannot be modified.");
    }
}
```

If even `Update` should not appear on audit logs, split `IWriteRepository<T>` further into `IAddRepository<T>` and `IUpdateRepository<T>`:

```csharp
public interface IAddRepository<T> where T : class
{
    void Add(T entity);
    void AddRange(IEnumerable<T> entities);
}

public interface IUpdateRepository<T> where T : class
{
    void Update(T entity);
}

// Recompose IWriteRepository from the granular pieces
public interface IWriteRepository<T> : IAddRepository<T>, IUpdateRepository<T>
    where T : class
{
}

// Now AuditLogRepository is truly clean -- zero unsupported methods
public class AuditLogRepository : IReadRepository<AuditLogEntry>, IAddRepository<AuditLogEntry>
{
    private readonly List<AuditLogEntry> _logs = new();

    public AuditLogEntry? GetById(int id) => _logs.FirstOrDefault(l => l.Id == id);
    public IEnumerable<AuditLogEntry> GetAll() => _logs.AsReadOnly();
    public IEnumerable<AuditLogEntry> Find(Expression<Func<AuditLogEntry, bool>> predicate)
        => _logs.AsQueryable().Where(predicate).ToList();
    public void Add(AuditLogEntry entity) => _logs.Add(entity);
    public void AddRange(IEnumerable<AuditLogEntry> entities) => _logs.AddRange(entities);
}
```

Now there is zero possibility of calling `Update` or `Delete` on audit logs -- those methods simply do not exist on the type. The compiler enforces the business rule.

Consumer code benefits from precise dependency declarations:

```csharp
// This service only reads -- it CANNOT accidentally modify data
public class ReportingService
{
    private readonly IReadRepository<AuditLogEntry> _auditLogs;

    public ReportingService(IReadRepository<AuditLogEntry> auditLogs)
    {
        _auditLogs = auditLogs;
    }

    public IEnumerable<AuditLogEntry> GetRecentActivity(int days)
    {
        var cutoff = DateTime.UtcNow.AddDays(-days);
        return _auditLogs.Find(log => log.Timestamp >= cutoff);
    }
}
```

---

## 🔹 ISP in .NET: Framework Examples

### Collection Interfaces -- IEnumerable vs ICollection vs IList

.NET's collection interface hierarchy is a textbook example of ISP in practice. Each level adds exactly the capabilities that level needs:

```
IEnumerable<T>              -- iterate (foreach)
    |
ICollection<T>              -- iterate + count + add/remove/contains
    |
IList<T>                    -- iterate + count + add/remove + index access
```

| Interface | What It Adds | Typical Use |
|---|---|---|
| `IEnumerable<T>` | `GetEnumerator()` | Any sequence you can iterate. LINQ methods accept this. |
| `ICollection<T>` | `Count`, `Add()`, `Remove()`, `Contains()`, `Clear()`, `IsReadOnly` | Need to know size or modify the collection. |
| `IList<T>` | `this[int index]`, `IndexOf()`, `Insert()`, `RemoveAt()` | Need random access by position. |

A method that only needs to iterate should accept `IEnumerable<T>`, not `IList<T>`. This way it works with arrays, lists, hash sets, LINQ query results, generator methods -- anything iterable. Accepting `IList<T>` would exclude hash sets and generators for no reason.

```csharp
// Good: accepts the narrowest interface needed
public static decimal CalculateTotal(IEnumerable<OrderLine> lines)
{
    return lines.Sum(l => l.Price * l.Quantity);
}

// Unnecessarily restrictive: excludes any non-list collection
public static decimal CalculateTotalBad(IList<OrderLine> lines)
{
    return lines.Sum(l => l.Price * l.Quantity);
}
```

### Read-Only Collection Interfaces

.NET provides read-only counterparts that embody ISP for consumers who should not modify data:

| Read-Write | Read-Only Counterpart |
|---|---|
| `ICollection<T>` | `IReadOnlyCollection<T>` |
| `IList<T>` | `IReadOnlyList<T>` |
| `IDictionary<TKey, TValue>` | `IReadOnlyDictionary<TKey, TValue>` |
| `ISet<T>` | `IReadOnlySet<T>` (.NET 5+) |

`List<T>` implements both `IList<T>` and `IReadOnlyList<T>`. When you expose a `List<T>` through `IReadOnlyList<T>`, the consumer cannot call `Add()`, `Remove()`, or `Clear()` -- those methods do not exist on the interface. This is ISP: the consumer depends only on the read capability it needs.

```csharp
public class OrderService
{
    private readonly List<Order> _orders = new();

    // External consumers get read-only access
    public IReadOnlyList<Order> Orders => _orders.AsReadOnly();

    // Internal methods can still modify
    public void AddOrder(Order order) => _orders.Add(order);
}
```

> [!tip] Practical Tip
> Default to the narrowest collection interface:
> - Only iterating? Use `IEnumerable<T>`
> - Need count? Use `IReadOnlyCollection<T>`
> - Need index access? Use `IReadOnlyList<T>`
> - Need mutation? Use `IList<T>` or `ICollection<T>`
>
> Start narrow, widen only when the method genuinely needs the extra capability.

### ASP.NET Core Examples

ASP.NET Core is an excellent example of ISP applied at the framework level. The framework authors deliberately split what could be one monolithic interface into many focused ones.

**`IApplicationBuilder` vs `IEndpointRouteBuilder`:**

`IApplicationBuilder` is focused solely on configuring the HTTP request [[Web Dev/3 - Concepts and Patterns/Pipeline|pipeline]] (middleware). It exposes methods like `Use()`, `Build()`, and properties like `ApplicationServices`. It knows nothing about routing.

`IEndpointRouteBuilder` is focused solely on mapping endpoints to routes. It exposes `MapGet()`, `MapPost()`, and the `DataSources` collection. It knows nothing about middleware ordering.

If these were merged into one interface, every middleware author would depend on routing methods they never call, and every endpoint mapper would depend on middleware methods they never call. By keeping them separate, a middleware component depends only on `IApplicationBuilder`, and a routing extension depends only on `IEndpointRouteBuilder`.

**`IServiceCollection` vs `IServiceProvider`:**

`IServiceCollection` is focused exclusively on service registration -- `AddSingleton()`, `AddScoped()`, `AddTransient()`. It does not resolve services. `IServiceProvider` is focused exclusively on service resolution -- `GetService()`, `GetRequiredService()`. It does not register services. This separation means registration code never depends on the resolution mechanism, and resolution code never depends on the registration API.

```csharp
// Registration: only needs IServiceCollection
public static void ConfigureServices(IServiceCollection services)
{
    services.AddScoped<IOrderService, OrderService>();
    services.AddSingleton<ICacheProvider, RedisCacheProvider>();
}

// Resolution: only needs IServiceProvider
public class OrderController
{
    private readonly IOrderService _orderService;

    public OrderController(IOrderService orderService)
    {
        // Resolved by the DI container -- controller never touches IServiceCollection
        _orderService = orderService;
    }
}
```

**`ILogger` vs `ILoggerFactory` vs `ILoggerProvider`:**

| Interface | Responsibility |
|---|---|
| `ILogger` | Write log messages (`Log()`, `LogInformation()`, `LogError()`, etc.) |
| `ILoggerFactory` | Create `ILogger` instances by category name |
| `ILoggerProvider` | Supply the underlying logging backend (console, file, Seq, etc.) |

- Application code depends on `ILogger` or `ILogger<T>` -- it only needs to write log messages. It never sees factory or provider concerns.
- Framework startup code depends on `ILoggerFactory` to create loggers -- it never writes log messages itself.
- Third-party logging libraries implement `ILoggerProvider` to plug in their backend -- they never write application log messages or create loggers for consumers.

If all three roles were merged into one interface, a simple controller that just wants to log a message would also depend on factory-creation methods and provider-registration methods it never uses. Every change to how providers register would force recompilation of application code that only logs messages.

```csharp
// Application code: depends ONLY on ILogger<T>
public class OrderService
{
    private readonly ILogger<OrderService> _logger;

    public OrderService(ILogger<OrderService> logger)
    {
        _logger = logger;
    }

    public void PlaceOrder(Order order)
    {
        _logger.LogInformation("Placing order {OrderId}", order.Id);
        // ... business logic ...
        _logger.LogInformation("Order {OrderId} placed successfully", order.Id);
    }
}

// Startup code: depends on ILoggerFactory / ILoggingBuilder
builder.Logging.AddConsole();
builder.Logging.AddSeq("https://seq.example.com");

// Third-party: implements ILoggerProvider
public class SeqLoggerProvider : ILoggerProvider
{
    public ILogger CreateLogger(string categoryName) { /* ... */ }
    public void Dispose() { /* ... */ }
}
```

---

## 🔹 ISP in C# Language Features

### Interface Composition

C# supports interface inheritance, making it natural to compose small interfaces into larger ones when needed:

```csharp
public interface IReadable<T>
{
    T? GetById(int id);
    IEnumerable<T> GetAll();
}

public interface IWritable<T>
{
    void Add(T entity);
    void Update(T entity);
}

// Composed interface for classes that need both
public interface IReadWrite<T> : IReadable<T>, IWritable<T>
{
}

// A service can depend on exactly what it needs:
public class ReportingService
{
    private readonly IReadable<Order> _orders;

    public ReportingService(IReadable<Order> orders)
    {
        // This service only reads -- it cannot accidentally modify data
        _orders = orders;
    }
}

public class OrderService
{
    private readonly IReadWrite<Order> _orders;

    public OrderService(IReadWrite<Order> orders)
    {
        // This service reads and writes
        _orders = orders;
    }
}
```

This pattern -- **start small, compose up** -- is the safest approach. Define minimal interfaces from the start and compose them when a client genuinely needs the full surface.

### Explicit Interface Implementation

When you are stuck with a fat interface you cannot change (from a third-party library, for example), explicit interface implementation lets you hide unwanted members from the class's public API:

```csharp
// Third-party interface you cannot modify
public interface ILegacyDataAccess
{
    DataTable Query(string sql);
    int Execute(string sql);
    void BulkInsert(DataTable table);
    void TruncateTable(string tableName);  // dangerous!
    void DropTable(string tableName);       // extremely dangerous!
}

// Your implementation hides the dangerous methods
public class SafeDataAccess : ILegacyDataAccess
{
    private readonly string _connectionString;

    public SafeDataAccess(string connectionString)
    {
        _connectionString = connectionString;
    }

    // These are public -- safe to call directly
    public DataTable Query(string sql)
    {
        Console.WriteLine($"Executing query: {sql}");
        return new DataTable();
    }

    public int Execute(string sql)
    {
        Console.WriteLine($"Executing command: {sql}");
        return 1;
    }

    public void BulkInsert(DataTable table)
    {
        Console.WriteLine($"Bulk inserting {table.Rows.Count} rows.");
    }

    // Explicit implementation -- not visible on SafeDataAccess directly,
    // only accessible through the ILegacyDataAccess interface reference
    void ILegacyDataAccess.TruncateTable(string tableName)
    {
        throw new NotSupportedException("Table truncation is disabled for safety.");
    }

    void ILegacyDataAccess.DropTable(string tableName)
    {
        throw new NotSupportedException("Table dropping is disabled for safety.");
    }
}

// Usage:
var access = new SafeDataAccess("Server=.;Database=MyDb;");
access.Query("SELECT 1");       // works fine
// access.TruncateTable("Users"); // COMPILE ERROR -- not visible on SafeDataAccess
// access.DropTable("Users");     // COMPILE ERROR -- not visible on SafeDataAccess

// Only accessible via explicit cast (deliberate and obvious):
ILegacyDataAccess legacy = access;
// legacy.TruncateTable("Users"); // compiles but throws NotSupportedException
```

> [!warning] Common Misconception
> Explicit interface implementation is a **workaround**, not a **solution**. It is a pragmatic tool when you cannot change the interface, but if you own the interface, the correct fix is to segregate it. Explicit implementation hides the problem from the class's public surface but does not remove it from the interface contract.

### Default Interface Methods (C# 8+)

C# 8 introduced default interface methods, which allow adding a method to an interface with a default implementation:

```csharp
public interface ILogger
{
    void Log(string message, LogLevel level);

    // Default implementations -- existing implementers don't break
    void LogInfo(string message) => Log(message, LogLevel.Information);
    void LogWarning(string message) => Log(message, LogLevel.Warning);
    void LogError(string message) => Log(message, LogLevel.Error);
}
```

Default interface methods can **mitigate** ISP violations by providing default no-op or basic implementations so that new members added to an interface do not force all implementers to update. However, they do not **solve** the underlying ISP problem. If an interface genuinely has methods that some implementers should not support, default methods just hide that design issue behind defaults.

Where default interface methods genuinely help ISP is **backward compatibility**: you can add a new convenience method with a sensible default without breaking existing implementers or creating a new interface version. This prevents the "interface versioning" problem where you end up with `IFoo`, `IFoo2`, `IFoo3`.

---

## 🔹 How to Identify ISP Violations

**1. `throw new NotImplementedException()` or `throw new NotSupportedException()` in interface implementations.** This is the most obvious red flag. If a class implements an interface but must throw for certain methods, the interface is too broad for that class.

**2. Empty method bodies (no-ops).** Just as bad as throwing, sometimes worse -- a no-op silently does nothing when the caller expected an action. The bug is invisible until someone audits the code.

```csharp
// Silent ISP violation -- Scan does nothing, caller has no idea
public void Scan(string document)
{
    // This printer doesn't support scanning
}
```

**3. Classes implementing interfaces but only using a fraction of the methods.** If a class implements an interface with 10 methods but only provides real logic for 2-3 of them, the interface is serving multiple roles and should be split.

**4. Mocking pain in unit tests.** When writing a test for a class that depends on an interface, if you find yourself writing `Mock<IFatInterface>` and setting up stubs for methods the class under test never calls, that is an ISP violation making your tests harder than they need to be:

```csharp
// Painful -- must set up methods the service never calls
var mock = new Mock<IWorker>();
mock.Setup(w => w.Work()).Verifiable();
mock.Setup(w => w.Eat());            // not used by WorkScheduler
mock.Setup(w => w.Sleep());          // not used by WorkScheduler
mock.Setup(w => w.AttendMeeting());  // not used by WorkScheduler
mock.Setup(w => w.WriteReport());    // not used by WorkScheduler
mock.Setup(w => w.SubmitTimesheet()); // not used by WorkScheduler

var scheduler = new WorkScheduler(mock.Object);

// Clean -- mock only what matters
var mock = new Mock<IWorkable>();
mock.Setup(w => w.Work()).Verifiable();
var scheduler = new WorkScheduler(mock.Object);
```

**5. "Interface pollution" -- the interface grows every sprint.** If the same interface gets a new method every iteration because different consumers need different things, it is becoming a dumping ground. Each new method forces all implementers to update, even those that have no use for the new behavior.

**6. IntelliSense clutter.** When you type `myService.` and IntelliSense shows 20+ methods, most of which are irrelevant to the current context, the interface is too broad. Segregated interfaces keep IntelliSense clean and contextual.

**7. The "header interface" smell.** If an interface was created by right-clicking a class and selecting "Extract Interface," capturing every public method, it is almost certainly a fat interface. Interfaces should be designed from the *consumer's* perspective, not the *provider's*.

---

## 🔹 ISP vs SRP

ISP and [[1 - Single Responsibility Principle|SRP]] are complementary but operate at different levels. Understanding the distinction prevents confusion:

| Aspect | SRP | ISP |
|---|---|---|
| **Applies to** | Classes (implementation) | Interfaces (contracts/APIs) |
| **Focus** | What a class **does** internally | What a class is **forced to expose** |
| **Core question** | "Does this class have one reason to change?" | "Does this interface force clients to depend on methods they don't use?" |
| **Violation symptom** | A class doing too many things | An implementer with empty/throwing methods |
| **Fix** | Split the class into smaller classes | Split the interface into smaller interfaces |
| **Perspective** | From the **implementer's** point of view | From the **client's** (consumer's) point of view |
| **Goal** | Cohesive implementation | Minimal dependency surface |
| **Granularity** | One responsibility per class | One role per interface |
| **Example violation** | A `UserService` that handles auth, email, and logging | An `IDevice` that forces a lamp to implement `PlayMusic()` |

A class can satisfy SRP (it has one cohesive responsibility) but still violate ISP (it implements an interface broader than its role). Conversely, an interface can satisfy ISP (small and focused) but the class implementing it can violate SRP (doing too many things behind that interface).

They complement each other: SRP keeps implementations focused, ISP keeps contracts focused. Together they ensure both sides of the abstraction boundary are clean.

> [!tip] Practical Tip
> When refactoring, ask both questions:
> - **SRP**: "Does this *class* have more than one reason to change?" If yes, split the class.
> - **ISP**: "Does this *interface* force any implementer to provide methods it doesn't need?" If yes, split the interface.
>
> Often fixing one reveals the other. A class with too many responsibilities (SRP) usually implements a fat interface (ISP), and vice versa.

---

## 🔹 Role Interfaces vs Header Interfaces

This distinction is fundamental to applying ISP correctly. It determines whether your interfaces serve consumers or merely mirror implementations.

### Header Interfaces

A **header interface** mirrors a class's public API one-to-one. It is extracted from the class after the fact, capturing everything the class does. This often leads to fat interfaces because the class's public API may serve multiple consumers with different needs.

```csharp
// Header interface -- mirrors the class, serves no one in particular
public interface IOrderProcessor
{
    void ValidateOrder(Order order);
    decimal CalculateTax(Order order);
    void ApplyDiscount(Order order, decimal percentage);
    void ProcessPayment(Order order, PaymentInfo payment);
    void SendConfirmationEmail(Order order);
    void UpdateInventory(Order order);
    void GenerateInvoice(Order order);
    void NotifyWarehouse(Order order);
}
```

This interface was created by extracting every public method from an `OrderProcessor` class. It serves as a "table of contents" for the class but does not represent what any specific consumer needs.

### Role Interfaces

A **role interface** captures what a specific consumer needs from a dependency. It is designed from the client's perspective: "What does this consumer need to do its job?"

```csharp
// Role interfaces -- each captures what a specific consumer needs
public interface IOrderValidator
{
    void ValidateOrder(Order order);
}

public interface ITaxCalculator
{
    decimal CalculateTax(Order order);
}

public interface IDiscountApplier
{
    void ApplyDiscount(Order order, decimal percentage);
}

public interface IPaymentProcessor
{
    void ProcessPayment(Order order, PaymentInfo payment);
}

public interface IOrderNotifier
{
    void SendConfirmationEmail(Order order);
    void NotifyWarehouse(Order order);
}

public interface IInvoiceGenerator
{
    void GenerateInvoice(Order order);
}

public interface IInventoryUpdater
{
    void UpdateInventory(Order order);
}
```

The guiding question is: **"What does the CLIENT need?"** -- not "What does the implementer provide?" Design interfaces from the consumer's perspective, not the provider's.

A single class can still implement all of these:

```csharp
public class OrderProcessor : IOrderValidator, ITaxCalculator, IDiscountApplier,
                               IPaymentProcessor, IOrderNotifier,
                               IInvoiceGenerator, IInventoryUpdater
{
    public void ValidateOrder(Order order) { /* ... */ }
    public decimal CalculateTax(Order order) { /* ... */ return 0; }
    public void ApplyDiscount(Order order, decimal percentage) { /* ... */ }
    public void ProcessPayment(Order order, PaymentInfo payment) { /* ... */ }
    public void SendConfirmationEmail(Order order) { /* ... */ }
    public void NotifyWarehouse(Order order) { /* ... */ }
    public void GenerateInvoice(Order order) { /* ... */ }
    public void UpdateInventory(Order order) { /* ... */ }
}
```

But each consumer depends only on the slice it needs:

```csharp
// CheckoutController only needs validation, tax, and payment
public class CheckoutController
{
    private readonly IOrderValidator _validator;
    private readonly ITaxCalculator _taxCalculator;
    private readonly IPaymentProcessor _paymentProcessor;

    public CheckoutController(
        IOrderValidator validator,
        ITaxCalculator taxCalculator,
        IPaymentProcessor paymentProcessor)
    {
        _validator = validator;
        _taxCalculator = taxCalculator;
        _paymentProcessor = paymentProcessor;
    }
}

// WarehouseService only needs inventory and notification
public class WarehouseService
{
    private readonly IInventoryUpdater _inventory;
    private readonly IOrderNotifier _notifier;

    public WarehouseService(IInventoryUpdater inventory, IOrderNotifier notifier)
    {
        _inventory = inventory;
        _notifier = notifier;
    }
}
```

---

## 🔹 How to Apply ISP: Refactoring Fat Interfaces

When you inherit a codebase with fat interfaces, refactoring them in a backward-compatible way follows these steps:

**Step 1: Identify the fat interface and audit its consumers.**

```csharp
// The fat interface
public interface IUserService
{
    User GetById(int id);
    User GetByEmail(string email);
    IEnumerable<User> GetAll();
    void Create(User user);
    void Update(User user);
    void Delete(int id);
    void SendWelcomeEmail(User user);
    void ResetPassword(string email);
    bool ValidateCredentials(string email, string password);
}
```

**Step 2: Group methods by which consumers use them.** Look at every class that depends on `IUserService` and note which methods each actually calls:

- `UserProfileController` calls `GetById`, `Update`
- `AdminDashboard` calls `GetAll`, `Delete`
- `AuthenticationService` calls `GetByEmail`, `ValidateCredentials`, `ResetPassword`
- `RegistrationService` calls `Create`, `SendWelcomeEmail`

**Step 3: Extract small interfaces from the groupings.**

```csharp
public interface IUserReader
{
    User GetById(int id);
    User GetByEmail(string email);
    IEnumerable<User> GetAll();
}

public interface IUserWriter
{
    void Create(User user);
    void Update(User user);
    void Delete(int id);
}

public interface IUserAuthenticator
{
    bool ValidateCredentials(string email, string password);
    void ResetPassword(string email);
}

public interface IUserNotifier
{
    void SendWelcomeEmail(User user);
}
```

**Step 4: Make the original fat interface extend the new ones (backward compatible).**

```csharp
// Existing code that depends on IUserService still compiles
public interface IUserService : IUserReader, IUserWriter,
                                 IUserAuthenticator, IUserNotifier
{
}
```

**Step 5: Gradually migrate consumers to depend on the narrow interfaces.**

```csharp
// Before:
public class UserProfileController
{
    private readonly IUserService _userService; // depends on everything
    public UserProfileController(IUserService userService)
    {
        _userService = userService;
    }
}

// After:
public class UserProfileController
{
    private readonly IUserReader _reader;   // depends only on what it needs
    private readonly IUserWriter _writer;

    public UserProfileController(IUserReader reader, IUserWriter writer)
    {
        _reader = reader;
        _writer = writer;
    }
}
```

**Step 6: Once all consumers are migrated, remove the fat interface.** The fat `IUserService` interface can be deleted (or kept as a convenience composition type if some DI registrations still benefit from it).

This approach is fully backward compatible at every step. No existing code breaks during the transition. Each consumer can be migrated independently, making it safe to do in small pull requests over time.

---

## 🔹 Common Anti-Patterns and Pitfalls

### Over-Segregation

ISP can be taken too far. An interface with a single method is not always better than one with three related methods. If three methods are always used together and represent a single cohesive capability, they belong in one interface:

```csharp
// Over-segregated -- these are always used together
public interface IUserNameReader { string GetFirstName(int userId); }
public interface IUserLastNameReader { string GetLastName(int userId); }
public interface IUserEmailReader { string GetEmail(int userId); }

// Better -- one cohesive interface for reading user profile data
public interface IUserProfileReader
{
    string GetFirstName(int userId);
    string GetLastName(int userId);
    string GetEmail(int userId);
}
```

The test: if a consumer that needs `GetFirstName` would also always need `GetLastName` and `GetEmail`, they belong together. Split only when consumers genuinely use different subsets.

### Creating Interfaces for Every Class

Not every class needs an interface. If a class has exactly one implementation and no foreseeable second one, and it is not a dependency that needs to be mocked in tests, adding an interface is ceremony without value:

```csharp
// Unnecessary: one implementation, no testing need, no polymorphism
public interface IDateTimeProvider
{
    DateTime Now { get; }
}

public class DateTimeProvider : IDateTimeProvider
{
    public DateTime Now => DateTime.UtcNow;
}

// Unless you actually need to mock time in tests -- then it's justified
```

> [!tip] Practical Tip
> Create an interface when you need at least one of:
> 1. **Multiple implementations** (strategy pattern, different backends)
> 2. **Test isolation** (mock external dependencies)
> 3. **Dependency inversion** (break a dependency cycle between assemblies)
>
> If none of these apply, the concrete class is sufficient.

### Forgetting DI Registration

When you split a fat interface into multiple small ones, do not forget to register all the small interfaces in your DI container. A common mistake:

```csharp
// Before: one registration covered everything
services.AddScoped<IUserService, UserService>();

// After: must register each interface separately
// (assuming UserService implements all of them)
services.AddScoped<IUserReader, UserService>();
services.AddScoped<IUserWriter, UserService>();
services.AddScoped<IUserAuthenticator, UserService>();
services.AddScoped<IUserNotifier, UserService>();

// Or register once and forward:
services.AddScoped<UserService>();
services.AddScoped<IUserReader>(sp => sp.GetRequiredService<UserService>());
services.AddScoped<IUserWriter>(sp => sp.GetRequiredService<UserService>());
services.AddScoped<IUserAuthenticator>(sp => sp.GetRequiredService<UserService>());
services.AddScoped<IUserNotifier>(sp => sp.GetRequiredService<UserService>());
```
