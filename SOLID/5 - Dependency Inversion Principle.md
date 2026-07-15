---
tags:
  - solid
  - dip
  - dependency-inversion
---

## 🔹 Definition

The Dependency Inversion Principle (DIP) is the fifth and final principle of [[SOLID Overview|SOLID]], introduced by Robert C. Martin. It consists of two formal statements:

**Part A:** "High-level modules should not depend on low-level modules. Both should depend on abstractions."

**Part B:** "Abstractions should not depend on details. Details should depend on abstractions."

**What "high-level" and "low-level" mean:**
- **High-level modules** contain the business logic, the policy, the rules that define what the application *does*. Example: `OrderService` that orchestrates placing an order.
- **Low-level modules** contain the implementation details, the mechanisms that carry out the work. Example: `SqlOrderRepository` that writes to a SQL database, `SmtpEmailSender` that sends emails via SMTP.

**What "abstractions" and "details" mean:**
- **Abstractions** are interfaces or abstract classes that define *what* operations are available without specifying *how* they are performed. Example: `IOrderRepository` with a `Save(Order order)` method.
- **Details** are concrete implementations that define *how* the operation is carried out. Example: `SqlOrderRepository` that uses `SqlConnection` and writes to a `dbo.Orders` table.

**The word "Inversion" is deliberate.** In traditional procedural and naive OOP design, the natural flow of dependency goes from high-level to low-level: the business logic directly references the database class, the file writer, the email sender. DIP *inverts* this direction. Instead of the business logic depending downward on infrastructure, both the business logic and the infrastructure depend on an abstraction that the business logic defines.

DIP is **not** simply "use interfaces." You can use interfaces everywhere and still violate DIP if the interface is defined in the wrong layer, owned by the wrong module, or merely mirrors a concrete implementation without thought. The principle is about the *direction* and *ownership* of dependencies.

## 🔹 Why It Matters

### Without DIP

Imagine `OrderService` directly creates and uses `SqlOrderRepository`:

- **Changing the database requires changing business logic.** If you migrate from SQL Server to PostgreSQL, you must modify `OrderService` even though the business rules haven't changed.
- **You cannot unit test in isolation.** Testing `OrderService` requires a running SQL Server instance, correct connection strings, seeded data, and cleanup after each test. Tests become slow, brittle, and environment-dependent.
- **You cannot swap implementations.** Want to log orders to a file during development? Want to use an in-memory store for integration tests? You must modify the `OrderService` class itself.
- **Parallel development is blocked.** The team working on business logic must wait for the team working on the database layer to finish, because the business logic is hard-coded to a specific database implementation.
- **Every low-level change ripples upward.** Renaming a method on `SqlOrderRepository`, changing its constructor parameters, or altering its namespace forces changes in `OrderService` and every other consumer.

### With DIP

`OrderService` depends on `IOrderRepository` (an abstraction it defines). `SqlOrderRepository` implements that interface:

- **Business logic is insulated from infrastructure changes.** Switching from SQL Server to MongoDB means writing a new `MongoOrderRepository` that implements `IOrderRepository`. `OrderService` is untouched.
- **Unit testing becomes trivial.** Pass in a mock or in-memory implementation of `IOrderRepository`. No database required. Tests run in milliseconds.
- **Implementations are swappable.** Development uses `InMemoryOrderRepository`, staging uses `SqlOrderRepository`, production uses `SqlOrderRepository` with a read replica -- all without modifying a single line of business logic.
- **Teams work in parallel.** One team defines the interface and builds business logic. Another team implements the interface against the database. They work independently.
- **The system is extensible.** Adding a caching layer means creating `CachedOrderRepository` that wraps `IOrderRepository` -- the [[2 - Open-Closed Principle|Open-Closed Principle]] and DIP working together.

### Architectural foundation

DIP is the foundation of [[Clean Architecture]], Hexagonal Architecture (Ports and Adapters), and Onion Architecture. All of these architectures share the same core idea: dependencies point inward toward the domain/business logic, and outer layers (UI, database, external services) depend on abstractions defined by the inner layers. Without DIP, none of these architectures are possible.

## 🔹 The Key Insight -- Who Owns the Interface?

This is the **most commonly misunderstood** aspect of DIP, and the part that separates developers who truly understand the principle from those who think DIP means "use interfaces."

### The traditional (violating) dependency direction

```
WITHOUT DIP (traditional layered architecture):

  OrderService ──depends on──► SqlOrderRepository
  (high-level)                  (low-level)

  The high-level module depends directly on the low-level module.
  The dependency arrow points DOWNWARD from policy to detail.
```

### The inverted dependency direction

```
WITH DIP:

  OrderService ──depends on──► IOrderRepository ◄──implements── SqlOrderRepository
  (high-level)                  (abstraction)                    (low-level)

  Both modules depend on the abstraction.
  The dependency arrow from SqlOrderRepository points UPWARD toward the abstraction.
  THIS is the "inversion."
```

### Where is `IOrderRepository` defined?

This is the critical question. Consider two scenarios:

**Scenario A (WRONG -- violates DIP):**
```
Project: DataAccess.dll
├── IOrderRepository.cs       <-- Interface defined here
├── SqlOrderRepository.cs     <-- Implementation here
└── ...

Project: Business.dll
├── OrderService.cs            <-- References DataAccess.dll to use IOrderRepository
└── ...
```

In this layout, `Business.dll` depends on `DataAccess.dll`. The high-level module still depends on the low-level module. You added an interface, but you did **not** invert the dependency. The interface is owned by the infrastructure layer, which means the infrastructure layer dictates the contract. If `DataAccess.dll` changes the interface, `Business.dll` must follow.

**Scenario B (CORRECT -- follows DIP):**
```
Project: Business.dll
├── Interfaces/
│   └── IOrderRepository.cs   <-- Interface defined HERE, in the business layer
├── Services/
│   └── OrderService.cs        <-- Uses IOrderRepository (same assembly)
└── ...

Project: DataAccess.dll
├── SqlOrderRepository.cs     <-- Implements IOrderRepository from Business.dll
└── ...                            References Business.dll
```

Now `DataAccess.dll` depends on `Business.dll` (to implement the interface defined there). The dependency arrow is **inverted**: low-level depends on high-level. The business layer *owns* the abstraction. The business layer defines *what* it needs. The infrastructure layer adapts to meet that contract.

### Why ownership matters

- The module that **defines** the interface **controls** the contract.
- The module that **implements** the interface **adapts** to the contract.
- If the business layer owns the interface, it defines operations in terms of business concepts (`Save(Order)`, `FindByCustomerId(int)`) rather than infrastructure concepts (`ExecuteStoredProcedure(string, SqlParameter[])`) .
- The business layer is free to evolve its interface as business requirements change, and infrastructure adapters follow.
- This is the literal "inversion": in traditional design, the business layer must adapt to whatever API the database layer exposes. With DIP, the database layer must adapt to whatever API the business layer demands.

## 🔹 Violation Example

Here is a realistic `OrderService` that violates DIP. Every line compiles, and every line demonstrates a problem:

```csharp
// All classes in the same project or with direct references to concrete types.
// This is the WRONG way to structure dependencies.

public class SqlOrderRepository
{
    private readonly string _connectionString;

    public SqlOrderRepository()
    {
        // Hard-coded connection string -- another problem on top of DIP violation
        _connectionString = "Server=localhost;Database=OrdersDb;Trusted_Connection=true;";
    }

    public void Save(Order order)
    {
        using var connection = new SqlConnection(_connectionString);
        connection.Open();

        using var command = new SqlCommand(
            "INSERT INTO Orders (Id, CustomerId, Total, CreatedAt) VALUES (@Id, @CustomerId, @Total, @CreatedAt)",
            connection);

        command.Parameters.AddWithValue("@Id", order.Id);
        command.Parameters.AddWithValue("@CustomerId", order.CustomerId);
        command.Parameters.AddWithValue("@Total", order.Total);
        command.Parameters.AddWithValue("@CreatedAt", order.CreatedAt);
        command.ExecuteNonQuery();
    }

    public Order? GetById(Guid id)
    {
        using var connection = new SqlConnection(_connectionString);
        connection.Open();

        using var command = new SqlCommand("SELECT * FROM Orders WHERE Id = @Id", connection);
        command.Parameters.AddWithValue("@Id", id);

        using var reader = command.ExecuteReader();
        if (!reader.Read()) return null;

        return new Order
        {
            Id = reader.GetGuid(reader.GetOrdinal("Id")),
            CustomerId = reader.GetInt32(reader.GetOrdinal("CustomerId")),
            Total = reader.GetDecimal(reader.GetOrdinal("Total")),
            CreatedAt = reader.GetDateTime(reader.GetOrdinal("CreatedAt"))
        };
    }
}

public class SmtpEmailSender
{
    public void SendOrderConfirmation(string to, Order order)
    {
        using var client = new SmtpClient("smtp.company.com", 587);
        client.Credentials = new NetworkCredential("orders@company.com", "password123");
        client.EnableSsl = true;

        var message = new MailMessage(
            "orders@company.com",
            to,
            $"Order Confirmation #{order.Id}",
            $"Your order for ${order.Total} has been placed.");

        client.Send(message);
    }
}

public class OrderService
{
    // VIOLATION: directly instantiating concrete low-level dependencies
    private readonly SqlOrderRepository _repository = new SqlOrderRepository();
    private readonly SmtpEmailSender _emailSender = new SmtpEmailSender();

    public void PlaceOrder(Order order, string customerEmail)
    {
        if (order.Total <= 0)
            throw new ArgumentException("Order total must be positive.");

        order.CreatedAt = DateTime.UtcNow;

        // VIOLATION: calling directly into infrastructure
        _repository.Save(order);
        _emailSender.SendOrderConfirmation(customerEmail, order);
    }

    public Order? GetOrder(Guid id)
    {
        return _repository.GetById(id);
    }
}
```

### Every problem with this code

| Problem | Explanation |
|---|---|
| `OrderService` creates `SqlOrderRepository` with `new` | `OrderService` controls the lifetime and creation of its dependencies. It is tightly coupled to the exact concrete type. |
| Cannot test without a real SQL Server | Calling `PlaceOrder` in a test will attempt to open a real database connection. If the database is down, misconfigured, or missing test data, the test fails for reasons unrelated to business logic. |
| Cannot test without a real SMTP server | `SendOrderConfirmation` will attempt to send a real email. Tests either fail or send spam. |
| Cannot swap database implementations | Switching to PostgreSQL, MongoDB, or an in-memory store requires modifying `OrderService` itself. |
| Cannot swap email providers | Migrating from SMTP to SendGrid, Amazon SES, or a queued email system requires modifying `OrderService`. |
| `OrderService` knows about SQL, SMTP, connection strings | The business logic layer has knowledge of infrastructure concerns it should not be aware of. |
| Hard-coded connection string | A secondary issue, but it compounds the coupling problem. |
| No way to add cross-cutting concerns | Want to add logging around repository calls? Caching? Retry logic? You must modify `OrderService` or `SqlOrderRepository` directly. |

## 🔹 Fixed Example

The fix involves three steps: define abstractions in the business layer, refactor the service to depend on abstractions, and create implementations in the infrastructure layer.

### Step 1: Define abstractions in the Business layer

```csharp
// ===== Business Layer (Business.dll) =====

// These interfaces are defined IN the business layer.
// They describe what the business logic needs, in business terms.

namespace Business.Interfaces
{
    /// <summary>
    /// Defines persistence operations for orders.
    /// Owned by the business layer -- infrastructure must adapt to this contract.
    /// </summary>
    public interface IOrderRepository
    {
        void Save(Order order);
        Order? GetById(Guid id);
        IReadOnlyList<Order> GetByCustomerId(int customerId);
    }

    /// <summary>
    /// Defines notification operations.
    /// Note: the interface uses business language ("notify order placed"),
    /// not infrastructure language ("send SMTP email").
    /// </summary>
    public interface INotificationService
    {
        void NotifyOrderPlaced(Order order, string customerEmail);
    }
}
```

### Step 2: Business service depends on abstractions

```csharp
// ===== Business Layer (Business.dll) =====

namespace Business.Services
{
    public class OrderService
    {
        private readonly IOrderRepository _repository;
        private readonly INotificationService _notificationService;

        // Dependencies are INJECTED through the constructor.
        // OrderService does not know or care what the concrete types are.
        public OrderService(IOrderRepository repository, INotificationService notificationService)
        {
            _repository = repository ?? throw new ArgumentNullException(nameof(repository));
            _notificationService = notificationService ?? throw new ArgumentNullException(nameof(notificationService));
        }

        public void PlaceOrder(Order order, string customerEmail)
        {
            // Pure business logic -- no infrastructure knowledge
            if (order.Total <= 0)
                throw new ArgumentException("Order total must be positive.", nameof(order));

            if (string.IsNullOrWhiteSpace(customerEmail))
                throw new ArgumentException("Customer email is required.", nameof(customerEmail));

            order.Id = Guid.NewGuid();
            order.CreatedAt = DateTime.UtcNow;
            order.Status = OrderStatus.Placed;

            _repository.Save(order);
            _notificationService.NotifyOrderPlaced(order, customerEmail);
        }

        public Order? GetOrder(Guid id)
        {
            if (id == Guid.Empty)
                throw new ArgumentException("Order ID cannot be empty.", nameof(id));

            return _repository.GetById(id);
        }

        public IReadOnlyList<Order> GetCustomerOrders(int customerId)
        {
            if (customerId <= 0)
                throw new ArgumentException("Customer ID must be positive.", nameof(customerId));

            return _repository.GetByCustomerId(customerId);
        }
    }
}
```

### Step 3: Infrastructure implements the abstractions

```csharp
// ===== Infrastructure Layer (Infrastructure.dll) =====
// This project references Business.dll to access the interfaces.
// The dependency arrow points FROM Infrastructure TO Business.

namespace Infrastructure.Persistence
{
    /// <summary>
    /// SQL Server implementation of IOrderRepository.
    /// Implements an interface it does NOT own -- the interface lives in Business.dll.
    /// </summary>
    public class SqlOrderRepository : IOrderRepository
    {
        private readonly string _connectionString;

        public SqlOrderRepository(string connectionString)
        {
            _connectionString = connectionString ?? throw new ArgumentNullException(nameof(connectionString));
        }

        public void Save(Order order)
        {
            using var connection = new SqlConnection(_connectionString);
            connection.Open();

            using var command = new SqlCommand(
                @"INSERT INTO Orders (Id, CustomerId, Total, CreatedAt, Status)
                  VALUES (@Id, @CustomerId, @Total, @CreatedAt, @Status)",
                connection);

            command.Parameters.AddWithValue("@Id", order.Id);
            command.Parameters.AddWithValue("@CustomerId", order.CustomerId);
            command.Parameters.AddWithValue("@Total", order.Total);
            command.Parameters.AddWithValue("@CreatedAt", order.CreatedAt);
            command.Parameters.AddWithValue("@Status", order.Status.ToString());
            command.ExecuteNonQuery();
        }

        public Order? GetById(Guid id)
        {
            using var connection = new SqlConnection(_connectionString);
            connection.Open();

            using var command = new SqlCommand(
                "SELECT Id, CustomerId, Total, CreatedAt, Status FROM Orders WHERE Id = @Id",
                connection);
            command.Parameters.AddWithValue("@Id", id);

            using var reader = command.ExecuteReader();
            if (!reader.Read()) return null;

            return MapOrder(reader);
        }

        public IReadOnlyList<Order> GetByCustomerId(int customerId)
        {
            var orders = new List<Order>();

            using var connection = new SqlConnection(_connectionString);
            connection.Open();

            using var command = new SqlCommand(
                "SELECT Id, CustomerId, Total, CreatedAt, Status FROM Orders WHERE CustomerId = @CustomerId ORDER BY CreatedAt DESC",
                connection);
            command.Parameters.AddWithValue("@CustomerId", customerId);

            using var reader = command.ExecuteReader();
            while (reader.Read())
            {
                orders.Add(MapOrder(reader));
            }

            return orders.AsReadOnly();
        }

        private static Order MapOrder(SqlDataReader reader)
        {
            return new Order
            {
                Id = reader.GetGuid(reader.GetOrdinal("Id")),
                CustomerId = reader.GetInt32(reader.GetOrdinal("CustomerId")),
                Total = reader.GetDecimal(reader.GetOrdinal("Total")),
                CreatedAt = reader.GetDateTime(reader.GetOrdinal("CreatedAt")),
                Status = Enum.Parse<OrderStatus>(reader.GetString(reader.GetOrdinal("Status")))
            };
        }
    }
}

namespace Infrastructure.Notifications
{
    /// <summary>
    /// SendGrid-based implementation of INotificationService.
    /// </summary>
    public class SendGridNotificationService : INotificationService
    {
        private readonly string _apiKey;
        private readonly string _fromEmail;

        public SendGridNotificationService(string apiKey, string fromEmail)
        {
            _apiKey = apiKey ?? throw new ArgumentNullException(nameof(apiKey));
            _fromEmail = fromEmail ?? throw new ArgumentNullException(nameof(fromEmail));
        }

        public void NotifyOrderPlaced(Order order, string customerEmail)
        {
            var client = new SendGridClient(_apiKey);
            var from = new EmailAddress(_fromEmail, "Order System");
            var to = new EmailAddress(customerEmail);
            var subject = $"Order Confirmation #{order.Id}";
            var body = $"Your order for {order.Total:C} has been placed on {order.CreatedAt:f}.";

            var msg = MailHelper.CreateSingleEmail(from, to, subject, body, htmlContent: null);
            client.SendEmailAsync(msg).GetAwaiter().GetResult();
        }
    }
}

namespace Infrastructure.Persistence
{
    /// <summary>
    /// In-memory implementation for testing and development.
    /// Demonstrates that swapping implementations is trivial when DIP is followed.
    /// </summary>
    public class InMemoryOrderRepository : IOrderRepository
    {
        private readonly List<Order> _orders = new();

        public void Save(Order order)
        {
            // Remove existing order with same ID if present (upsert behavior)
            _orders.RemoveAll(o => o.Id == order.Id);
            _orders.Add(order);
        }

        public Order? GetById(Guid id)
        {
            return _orders.FirstOrDefault(o => o.Id == id);
        }

        public IReadOnlyList<Order> GetByCustomerId(int customerId)
        {
            return _orders
                .Where(o => o.CustomerId == customerId)
                .OrderByDescending(o => o.CreatedAt)
                .ToList()
                .AsReadOnly();
        }
    }
}
```

### The domain model (shared)

```csharp
// ===== Business Layer (Business.dll) =====

namespace Business.Models
{
    public class Order
    {
        public Guid Id { get; set; }
        public int CustomerId { get; set; }
        public decimal Total { get; set; }
        public DateTime CreatedAt { get; set; }
        public OrderStatus Status { get; set; }
    }

    public enum OrderStatus
    {
        Placed,
        Processing,
        Shipped,
        Delivered,
        Cancelled
    }
}
```

## 🔹 DIP vs Dependency Injection (DI) -- Critical Distinction

This is where most developers get confused. These are **three separate concepts** that are related but not interchangeable:

### Dependency Inversion Principle (DIP)

A **design principle** that states high-level modules should not depend on low-level modules; both should depend on abstractions. DIP is about the *direction* of source-code dependency. It tells you *what* your dependency graph should look like, not *how* to achieve it. DIP is a compile-time, architectural concern.

### Dependency Injection (DI)

A **design pattern/technique** for providing an object with its dependencies from the outside rather than having the object create them internally. DI is one *mechanism* for achieving DIP, but it is not the only one. DI answers the question: "how does `OrderService` get its `IOrderRepository`?"

### Inversion of Control (IoC)

A **broader design principle** where the flow of control is inverted compared to traditional procedural programming. Instead of your code calling a framework, the framework calls your code. DI is one form of IoC. Event-driven programming is another. The Template Method pattern is another.

### IoC Container

A **tool/framework** that automates dependency injection. Examples: `Microsoft.Extensions.DependencyInjection`, Autofac, Ninject, Castle Windsor, Simple Injector. The container reads configuration (or conventions), constructs object graphs, manages lifetimes, and resolves dependencies automatically.

### You can have DI without DIP

```csharp
// DI is happening (dependency passed via constructor),
// but DIP is VIOLATED (depending on a concrete class, not an abstraction).
public class OrderService
{
    private readonly SqlOrderRepository _repository;

    // This IS dependency injection -- the dependency is injected from outside.
    // But this is NOT DIP -- we depend on a concrete class, not an abstraction.
    public OrderService(SqlOrderRepository repository)
    {
        _repository = repository;
    }
}
```

The dependency is injected (DI), but `OrderService` still depends directly on `SqlOrderRepository` (violates DIP). You cannot swap the implementation without modifying `OrderService`.

### You can have DIP without DI

```csharp
// DIP is followed (depending on abstraction),
// but DI is NOT used (dependency is resolved internally via factory).
public class OrderService
{
    private readonly IOrderRepository _repository;

    public OrderService()
    {
        // Using a factory to get the implementation -- not DI (nothing is injected).
        // But DIP IS satisfied: OrderService depends on IOrderRepository, not a concrete class.
        _repository = OrderRepositoryFactory.Create();
    }
}

public static class OrderRepositoryFactory
{
    public static IOrderRepository Create()
    {
        var config = ConfigurationManager.AppSettings["RepositoryType"];
        return config switch
        {
            "Sql" => new SqlOrderRepository(ConfigurationManager.ConnectionStrings["Default"].ConnectionString),
            "InMemory" => new InMemoryOrderRepository(),
            _ => throw new InvalidOperationException($"Unknown repository type: {config}")
        };
    }
}
```

### Comparison table

| Aspect | DIP | DI | IoC Container |
|---|---|---|---|
| **What it is** | Design principle | Design pattern / technique | Framework / tool |
| **Level** | Architectural (compile-time dependency direction) | Object construction (runtime wiring) | Automation of DI |
| **Concern** | Which module depends on which? | How does an object receive its dependencies? | Who builds and manages the object graph? |
| **Defined by** | Robert C. Martin (SOLID) | Martin Fowler and others | Various framework authors |
| **Example** | `OrderService` depends on `IOrderRepository`, not `SqlOrderRepository` | `OrderService` receives `IOrderRepository` via its constructor | `services.AddScoped<IOrderRepository, SqlOrderRepository>()` |
| **Can exist without the others?** | Yes (use factories, service locators) | Yes (inject concrete classes) | No (requires DI pattern to function) |
| **Answers the question** | "What should my dependency graph look like?" | "How do I provide dependencies to an object?" | "How do I automate the wiring?" |

## 🔹 Dependency Injection Patterns in C#

There are three recognized patterns for injecting dependencies. Each has specific use cases.

### Constructor injection (preferred)

Dependencies are passed through the constructor and stored as `readonly` fields.

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly INotificationService _notificationService;
    private readonly ILogger<OrderService> _logger;

    // All required dependencies declared in the constructor.
    // The class CANNOT be instantiated without providing them.
    public OrderService(
        IOrderRepository repository,
        INotificationService notificationService,
        ILogger<OrderService> logger)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
        _notificationService = notificationService ?? throw new ArgumentNullException(nameof(notificationService));
        _logger = logger ?? throw new ArgumentNullException(nameof(logger));
    }

    public void PlaceOrder(Order order, string customerEmail)
    {
        _logger.LogInformation("Placing order for customer {CustomerId}", order.CustomerId);

        order.Id = Guid.NewGuid();
        order.CreatedAt = DateTime.UtcNow;
        order.Status = OrderStatus.Placed;

        _repository.Save(order);
        _notificationService.NotifyOrderPlaced(order, customerEmail);

        _logger.LogInformation("Order {OrderId} placed successfully", order.Id);
    }
}
```

**Why constructor injection is preferred:**
- Makes dependencies **explicit**. You can see everything a class needs by looking at its constructor.
- Enforces **required** dependencies. The object cannot be created in an invalid state.
- Fields can be `readonly`, guaranteeing **immutability** after construction.
- Works naturally with IoC containers -- they resolve constructor parameters automatically.
- Makes **over-injection visible**. If the constructor has 8 parameters, it is an obvious signal that the class has too many responsibilities (violation of the [[1 - Single Responsibility Principle]]).

### Property / setter injection

Dependencies are assigned through public settable properties after construction.

```csharp
public class ReportGenerator
{
    private readonly IReportDataSource _dataSource;

    // Required dependency -- still via constructor
    public ReportGenerator(IReportDataSource dataSource)
    {
        _dataSource = dataSource ?? throw new ArgumentNullException(nameof(dataSource));
    }

    // Optional dependency -- via property injection.
    // Has a sensible default, but can be overridden.
    public IReportFormatter Formatter { get; set; } = new PlainTextReportFormatter();

    // Optional dependency -- caching is not required for the class to function.
    public ICache? Cache { get; set; }

    public string GenerateReport(int year, int quarter)
    {
        var cacheKey = $"report-{year}-Q{quarter}";

        // Use cache if available, but function correctly without it
        if (Cache is not null)
        {
            var cached = Cache.Get<string>(cacheKey);
            if (cached is not null) return cached;
        }

        var data = _dataSource.GetData(year, quarter);
        var report = Formatter.Format(data);

        Cache?.Set(cacheKey, report, TimeSpan.FromHours(1));

        return report;
    }
}

// Usage:
var generator = new ReportGenerator(dataSource);
// Optionally override the formatter:
generator.Formatter = new HtmlReportFormatter();
// Optionally provide a cache:
generator.Cache = new RedisCache(connectionString);
```

**When to use property injection:**
- The dependency is truly **optional** -- the class functions correctly without it.
- There is a **sensible default** implementation.
- You want to allow **post-construction configuration** (common in framework code).

**Risks:**
- The object can exist in a partially configured state.
- The dependency can be changed after construction, breaking immutability.
- Less discoverable -- you must know to set the property.

### Method injection

Dependencies are passed as parameters to the method that uses them.

```csharp
public class PriceCalculator
{
    public decimal CalculateTotal(
        Order order,
        ITaxCalculator taxCalculator,    // Injected per call -- may vary per request
        IDiscountStrategy discountStrategy) // Injected per call -- may vary per customer
    {
        var subtotal = order.Items.Sum(item => item.Price * item.Quantity);
        var discount = discountStrategy.CalculateDiscount(order, subtotal);
        var afterDiscount = subtotal - discount;
        var tax = taxCalculator.CalculateTax(afterDiscount, order.ShippingAddress);

        return afterDiscount + tax;
    }
}

// Usage -- different strategies for different calls:
var calculator = new PriceCalculator();

// US customer with holiday discount
var usTotal = calculator.CalculateTotal(
    usOrder,
    new UsTaxCalculator(),
    new HolidayDiscountStrategy());

// EU customer with loyalty discount
var euTotal = calculator.CalculateTotal(
    euOrder,
    new EuVatCalculator(),
    new LoyaltyDiscountStrategy(customerTier));
```

**When to use method injection:**
- The dependency **varies per call**, not per instance.
- The dependency is a **strategy** that the caller selects (Strategy pattern).
- The dependency carries **request-scoped context** that isn't known at construction time.

### Comparison table

| Pattern | When to Use | Pros | Cons |
|---|---|---|---|
| **Constructor** | Required dependencies that the class needs for its entire lifetime | Explicit, enforceable, immutable, IoC-friendly | Constructor can grow large (but that signals SRP violation) |
| **Property** | Optional dependencies with sensible defaults | Flexible, supports optional features | Mutable, partially initialized state possible, less discoverable |
| **Method** | Per-call dependencies that vary between invocations | Maximum flexibility, caller controls strategy | Caller burden increases, not suitable for dependencies used across many methods |

## 🔹 DIP in ASP.NET Core

ASP.NET Core has DIP and DI built into its DNA. The entire framework is designed around these principles.

### IServiceCollection registration

All dependency registrations happen in `Program.cs` (or `Startup.cs` in older templates). This is the **Composition Root** -- the single place in the application where the entire object graph is wired together.

```csharp
// Program.cs -- the Composition Root

var builder = WebApplication.CreateBuilder(args);

// Register business-layer abstractions with infrastructure implementations.
// The interface (left) is defined in the Business layer.
// The implementation (right) is defined in the Infrastructure layer.
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
builder.Services.AddScoped<INotificationService, SendGridNotificationService>();
builder.Services.AddScoped<IOrderService, OrderService>();

// Register with a factory when construction is complex
builder.Services.AddScoped<IPaymentGateway>(provider =>
{
    var config = provider.GetRequiredService<IConfiguration>();
    var apiKey = config["Payment:ApiKey"]
        ?? throw new InvalidOperationException("Payment API key not configured.");
    return new StripePaymentGateway(apiKey);
});

// Register infrastructure services
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));

var app = builder.Build();

// ... middleware pipeline ...

app.Run();
```

### Service lifetimes

ASP.NET Core's DI container supports three lifetimes. Choosing the wrong one causes subtle, hard-to-debug issues.

| Lifetime | Registration | Created | Disposed | When to Use | Gotchas |
|---|---|---|---|---|---|
| **Transient** | `AddTransient<I, T>()` | Every time it is requested | When the scope ends | Lightweight, stateless services. Services that should not share state between consumers. | Can cause excessive allocations if requested frequently. Each consumer gets its own instance, so state is never shared. |
| **Scoped** | `AddScoped<I, T>()` | Once per HTTP request (scope) | When the request ends | Services that should share state within a single request but not across requests. Database contexts (`DbContext`), unit-of-work, repositories. | **Captive dependency problem**: a Singleton that depends on a Scoped service captures a stale instance. The Scoped service is disposed after the first request, but the Singleton keeps a reference to the dead instance. ASP.NET Core throws `InvalidOperationException` by default to prevent this. |
| **Singleton** | `AddSingleton<I, T>()` | Once for the entire application lifetime | When the application shuts down | Thread-safe, stateless services. Caches, configuration wrappers, HTTP client factories. | Must be **thread-safe**. All requests share the same instance. Memory is held for the application lifetime. Cannot depend on Scoped or Transient services (captive dependency). |

**The captive dependency rule (critical to remember):**
A service should only depend on services with an **equal or longer** lifetime.

```
Singleton can depend on:    Singleton only
Scoped can depend on:       Singleton, Scoped
Transient can depend on:    Singleton, Scoped, Transient
```

Violating this causes a Scoped or Transient service to be "captured" inside a longer-lived service, keeping it alive past its intended lifetime. In development, ASP.NET Core's `ValidateScopes` option (enabled by default) throws an exception when this happens.

### Built-in DIP examples in ASP.NET Core

The framework itself follows DIP extensively. You interact with abstractions, not concrete implementations:

```csharp
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;
    private readonly ILogger<OrdersController> _logger;
    private readonly IConfiguration _configuration;
    private readonly IOptions<SmtpSettings> _smtpSettings;

    // All of these are abstractions provided by the framework.
    // You never see the concrete implementations.
    public OrdersController(
        IOrderService orderService,            // Your own abstraction
        ILogger<OrdersController> logger,      // Framework abstraction (could be Console, File, Seq, etc.)
        IConfiguration configuration,          // Framework abstraction (could be JSON, env vars, Azure Key Vault)
        IOptions<SmtpSettings> smtpSettings)   // Framework abstraction (strongly-typed config)
    {
        _orderService = orderService;
        _logger = logger;
        _configuration = configuration;
        _smtpSettings = smtpSettings;
    }

    [HttpPost]
    public IActionResult PlaceOrder([FromBody] PlaceOrderRequest request)
    {
        _logger.LogInformation("Placing order for customer {CustomerId}", request.CustomerId);

        var order = new Order
        {
            CustomerId = request.CustomerId,
            Total = request.Total
        };

        _orderService.PlaceOrder(order, request.Email);

        return CreatedAtAction(nameof(GetOrder), new { id = order.Id }, order);
    }

    [HttpGet("{id:guid}")]
    public IActionResult GetOrder(Guid id)
    {
        var order = _orderService.GetOrder(id);
        return order is not null ? Ok(order) : NotFound();
    }
}
```

### The Composition Root concept

The Composition Root is the **single location** in the application where the entire object graph is assembled. In ASP.NET Core, this is `Program.cs`. Key rules:

- **Only the Composition Root should reference the IoC container.** Business logic, controllers, and services should never call `IServiceProvider.GetService<T>()`.
- **Only the Composition Root should reference concrete implementations.** This is the one place where `Business.dll` and `Infrastructure.dll` are both referenced.
- **The Composition Root is in the outermost layer** (the application entry point), not in the business or infrastructure layers.

### Service Locator anti-pattern

The Service Locator pattern is the **opposite** of constructor injection. Instead of declaring dependencies in the constructor, a class reaches into a global container to pull out whatever it needs. This is widely considered an anti-pattern.

```csharp
// ANTI-PATTERN: Service Locator
public class OrderService
{
    private readonly IServiceProvider _serviceProvider;

    public OrderService(IServiceProvider serviceProvider)
    {
        _serviceProvider = serviceProvider;
    }

    public void PlaceOrder(Order order, string customerEmail)
    {
        // Dependencies are HIDDEN -- you cannot tell what this class needs
        // by looking at its constructor.
        var repository = _serviceProvider.GetRequiredService<IOrderRepository>();
        var notifier = _serviceProvider.GetRequiredService<INotificationService>();

        order.Id = Guid.NewGuid();
        order.CreatedAt = DateTime.UtcNow;

        repository.Save(order);
        notifier.NotifyOrderPlaced(order, customerEmail);
    }
}
```

**Why Service Locator is an anti-pattern:**

| Problem | Explanation |
|---|---|
| **Hidden dependencies** | You cannot see what the class needs by looking at its constructor. You must read the entire implementation to discover dependencies. |
| **Harder to test** | You must set up a full `IServiceProvider` with all required registrations instead of simply passing mocks. |
| **Runtime failures instead of compile-time** | If a dependency is not registered, you get a runtime exception when the method is called, not a compile-time error or a startup-time error. |
| **Violates Explicit Dependencies Principle** | A class should explicitly state what it needs to do its job. |
| **Makes refactoring dangerous** | Adding or removing a `GetService` call deep in a method has no visible effect on the class's public API, so other developers may not realize the dependency has changed. |

**The one exception:** Factory patterns or middleware where you genuinely need to resolve services lazily or per-request within the Composition Root itself. Even then, prefer `IServiceScopeFactory` over raw `IServiceProvider`.

## 🔹 Layer Architecture with DIP

### Dependency direction

```
┌──────────────────────────────────────────────┐
│         Presentation Layer                   │
│   (Controllers, ViewModels, API Endpoints)   │
│                                              │
│   Depends on: Business Layer                 │
└──────────────────────┬───────────────────────┘
                       │ depends on (references)
                       ▼
┌──────────────────────────────────────────────┐
│           Business Layer                     │
│   (Services, Domain Models, Interfaces)      │
│                                              │
│   IOrderRepository defined HERE              │
│   INotificationService defined HERE          │
│   OrderService defined HERE                  │
└──────────────────────────────────────────────┘
                       ▲
                       │ implements (references)
┌──────────────────────┴───────────────────────┐
│         Infrastructure Layer                 │
│   (Repositories, Email, File I/O, Caching)   │
│                                              │
│   SqlOrderRepository implements              │
│       IOrderRepository                       │
│   SendGridNotificationService implements     │
│       INotificationService                   │
│                                              │
│   Depends on: Business Layer                 │
└──────────────────────────────────────────────┘
```

Notice: **Infrastructure depends on Business, not the other way around.** This is the inversion. In traditional layered architecture, Business would depend on Infrastructure (Data Access Layer). With DIP, the arrow flips.

### Realistic project folder structure

```
MyApp/
├── src/
│   ├── MyApp.Business/                        <-- Core business logic (no infrastructure refs)
│   │   ├── MyApp.Business.csproj
│   │   ├── Interfaces/
│   │   │   ├── IOrderRepository.cs
│   │   │   ├── ICustomerRepository.cs
│   │   │   ├── INotificationService.cs
│   │   │   └── IPaymentGateway.cs
│   │   ├── Models/
│   │   │   ├── Order.cs
│   │   │   ├── Customer.cs
│   │   │   └── OrderStatus.cs
│   │   └── Services/
│   │       ├── OrderService.cs
│   │       └── CustomerService.cs
│   │
│   ├── MyApp.Infrastructure/                  <-- References MyApp.Business
│   │   ├── MyApp.Infrastructure.csproj
│   │   ├── Persistence/
│   │   │   ├── AppDbContext.cs
│   │   │   ├── SqlOrderRepository.cs
│   │   │   └── SqlCustomerRepository.cs
│   │   ├── Notifications/
│   │   │   ├── SendGridNotificationService.cs
│   │   │   └── ConsoleNotificationService.cs  <-- Development fallback
│   │   └── Payment/
│   │       └── StripePaymentGateway.cs
│   │
│   └── MyApp.Api/                             <-- References both (Composition Root)
│       ├── MyApp.Api.csproj
│       ├── Program.cs                         <-- Wires everything together
│       └── Controllers/
│           ├── OrdersController.cs
│           └── CustomersController.cs
│
└── tests/
    ├── MyApp.Business.Tests/                  <-- Only references MyApp.Business
    │   ├── OrderServiceTests.cs
    │   └── CustomerServiceTests.cs
    └── MyApp.Integration.Tests/               <-- References all projects
        └── OrdersApiTests.cs
```

**Project reference directions (critical):**
- `MyApp.Business.csproj` -- references **nothing** (no project references)
- `MyApp.Infrastructure.csproj` -- references `MyApp.Business`
- `MyApp.Api.csproj` -- references `MyApp.Business` and `MyApp.Infrastructure`
- `MyApp.Business.Tests.csproj` -- references `MyApp.Business` only (tests business logic in isolation)

## 🔹 DIP Enables

### Unit testing with mocks

Because `OrderService` depends on abstractions, you can substitute test doubles in unit tests. Here is an example using the Moq library:

```csharp
using Moq;
using Xunit;

public class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _mockRepository;
    private readonly Mock<INotificationService> _mockNotificationService;
    private readonly OrderService _sut; // System Under Test

    public OrderServiceTests()
    {
        _mockRepository = new Mock<IOrderRepository>();
        _mockNotificationService = new Mock<INotificationService>();
        _sut = new OrderService(_mockRepository.Object, _mockNotificationService.Object);
    }

    [Fact]
    public void PlaceOrder_ValidOrder_SavesOrderAndNotifiesCustomer()
    {
        // Arrange
        var order = new Order { CustomerId = 1, Total = 99.99m };
        var email = "customer@example.com";

        // Act
        _sut.PlaceOrder(order, email);

        // Assert
        Assert.NotEqual(Guid.Empty, order.Id);                   // ID was assigned
        Assert.Equal(OrderStatus.Placed, order.Status);          // Status was set
        _mockRepository.Verify(r => r.Save(order), Times.Once); // Repository was called
        _mockNotificationService.Verify(
            n => n.NotifyOrderPlaced(order, email),
            Times.Once);                                         // Notification was sent
    }

    [Fact]
    public void PlaceOrder_ZeroTotal_ThrowsArgumentException()
    {
        // Arrange
        var order = new Order { CustomerId = 1, Total = 0m };

        // Act & Assert
        var ex = Assert.Throws<ArgumentException>(() => _sut.PlaceOrder(order, "test@test.com"));
        Assert.Contains("positive", ex.Message);

        // Verify repository was NEVER called -- order should be rejected before saving
        _mockRepository.Verify(r => r.Save(It.IsAny<Order>()), Times.Never);
    }

    [Fact]
    public void GetOrder_ExistingOrder_ReturnsOrder()
    {
        // Arrange
        var orderId = Guid.NewGuid();
        var expectedOrder = new Order { Id = orderId, CustomerId = 1, Total = 50m };
        _mockRepository.Setup(r => r.GetById(orderId)).Returns(expectedOrder);

        // Act
        var result = _sut.GetOrder(orderId);

        // Assert
        Assert.NotNull(result);
        Assert.Equal(orderId, result!.Id);
    }

    [Fact]
    public void GetOrder_NonExistentOrder_ReturnsNull()
    {
        // Arrange
        _mockRepository.Setup(r => r.GetById(It.IsAny<Guid>())).Returns((Order?)null);

        // Act
        var result = _sut.GetOrder(Guid.NewGuid());

        // Assert
        Assert.Null(result);
    }
}
```

These tests run in milliseconds with no database, no network, no SMTP server. This is only possible because DIP decoupled the business logic from infrastructure.

### Swapping implementations

DIP makes implementation swapping a configuration change, not a code change:

```csharp
// Program.cs -- swap implementations based on environment

if (builder.Environment.IsDevelopment())
{
    // Development: in-memory storage, console notifications
    builder.Services.AddSingleton<IOrderRepository, InMemoryOrderRepository>();
    builder.Services.AddSingleton<INotificationService, ConsoleNotificationService>();
}
else
{
    // Production: SQL Server, SendGrid
    builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
    builder.Services.AddScoped<INotificationService, SendGridNotificationService>();
}
```

### Architectural patterns

DIP is the enabling principle behind:
- **[[Clean Architecture]]** -- domain at the center, dependencies point inward
- **Hexagonal Architecture (Ports and Adapters)** -- interfaces are "ports," implementations are "adapters"
- **Plugin architectures** -- load implementations at runtime via reflection or configuration

## 🔹 Common Mistakes

### Over-abstraction: creating interfaces for everything

```csharp
// WRONG: interface for a class with no reason to ever be swapped
public interface IOrderValidator
{
    bool IsValid(Order order);
}

public class OrderValidator : IOrderValidator
{
    public bool IsValid(Order order)
    {
        return order.Total > 0 && order.CustomerId > 0;
    }
}
```

If `OrderValidator` contains pure business logic with no infrastructure dependencies, no I/O, and no reason to ever have a second implementation, the interface adds complexity without value. Just use the concrete class directly.

**Rule of thumb:** Create an interface when the class crosses an architectural boundary (I/O, network, file system), when you need polymorphism (multiple implementations), or when you need testability that mocking a concrete class cannot provide.

### Header interfaces: 1:1 mirrors of a single implementation

```csharp
// SMELL: interface that is a carbon copy of the class's public API
public interface IOrderService
{
    void PlaceOrder(Order order, string email);
    Order? GetOrder(Guid id);
    IReadOnlyList<Order> GetCustomerOrders(int customerId);
    void CancelOrder(Guid id);
    void UpdateOrderStatus(Guid id, OrderStatus status);
    decimal GetCustomerTotalSpend(int customerId);
    IReadOnlyList<Order> SearchOrders(string query, DateTime? from, DateTime? to);
}
```

This interface has 7 methods that exactly mirror one class. No other implementation will ever implement all 7 methods identically. This is a "header interface" -- it exists only because someone said "everything needs an interface." Instead, consider whether you actually need the interface, and if you do, whether it should be broken into smaller, focused interfaces per the [[4 - Interface Segregation Principle]].

### Abstracting stable dependencies

```csharp
// UNNECESSARY: wrapping stable framework types
public interface IDateTimeProvider
{
    DateTime UtcNow { get; }
}

public class SystemDateTimeProvider : IDateTimeProvider
{
    public DateTime UtcNow => DateTime.UtcNow;
}
```

`DateTime.UtcNow` is a stable, well-understood framework API. Abstracting it adds a layer of indirection that serves no purpose *unless* you specifically need to control time in tests (e.g., testing expiration logic). If you do need it, the abstraction is justified. If you don't, it's ceremony.

**Ask:** "Will I ever have a second implementation of this interface? Will I ever need to mock this in a test?" If both answers are no, skip the abstraction.

### Service Locator instead of constructor injection

Covered in detail in the ASP.NET Core section above. Prefer constructor injection. Reserve `IServiceProvider` for factory scenarios and middleware in the Composition Root.

### Circular dependencies

Circular dependencies happen when two classes depend on each other, directly or transitively:

```csharp
// CIRCULAR: OrderService needs IInventoryService, InventoryService needs IOrderService
public class OrderService
{
    private readonly IInventoryService _inventoryService;
    public OrderService(IInventoryService inventoryService) { _inventoryService = inventoryService; }
}

public class InventoryService
{
    private readonly IOrderService _orderService;
    public InventoryService(IOrderService orderService) { _orderService = orderService; }
}
// The IoC container will throw at startup: circular dependency detected.
```

**How to fix circular dependencies:**

1. **Extract the shared concern into a third service.** If both need each other, they probably share a responsibility that should be its own class.
2. **Use events/mediator.** Instead of `OrderService` calling `InventoryService` directly, have `OrderService` publish an `OrderPlaced` event that `InventoryService` subscribes to.
3. **Use `Lazy<T>` as a last resort.** Inject `Lazy<IOrderService>` to defer resolution, breaking the cycle. This is a workaround, not a fix -- it means your design still has a circular relationship.

### Constructor over-injection

```csharp
// SMELL: too many constructor parameters
public class OrderService
{
    public OrderService(
        IOrderRepository orderRepository,
        ICustomerRepository customerRepository,
        IInventoryService inventoryService,
        IPaymentGateway paymentGateway,
        INotificationService notificationService,
        ITaxCalculator taxCalculator,
        IShippingCalculator shippingCalculator,
        IDiscountService discountService,
        IAuditLogger auditLogger,
        IFeatureFlagService featureFlags)
    {
        // 10 dependencies -- this class does too much
    }
}
```

When a constructor has more than 3-4 dependencies, it is a strong signal that the class violates the [[1 - Single Responsibility Principle]]. The fix is not to hide the dependencies (e.g., by bundling them into a "parameter object" or using Service Locator). The fix is to **decompose the class** into smaller, focused classes, each with fewer responsibilities and dependencies.

## 🔹 When NOT to Apply DIP

DIP is a principle, not a law. Applying it everywhere adds unnecessary abstraction and complexity. Here is when to skip it:

**Stable, unlikely-to-change dependencies:**
`System.Math`, `System.String`, `System.Collections.Generic.List<T>`, `System.Text.Json.JsonSerializer` -- these are stable framework APIs. Wrapping them in interfaces adds indirection with zero benefit.

**Simple applications:**
A console tool that reads a CSV and writes a report does not need `IFileReader`, `ICsvParser`, and `IReportWriter`. The overhead of defining interfaces, wiring up DI, and creating separate assemblies exceeds the benefit when the application is 200 lines of code.

**Internal implementation details:**
A private helper method inside a class, or a small utility class used by a single service, does not need an interface. If it has no consumers outside its immediate context, abstracting it adds complexity without enabling any flexibility.

**Value objects and DTOs:**
`Order`, `Customer`, `Address`, `PlaceOrderRequest` -- these are data carriers. They don't perform I/O, don't have behavior that varies, and don't need to be swapped. Never create `IOrder` or `ICustomer`.

**YAGNI (You Aren't Gonna Need It):**
If there is exactly one implementation today, no foreseeable second implementation, and no testing difficulty, don't create an interface preemptively. You can always extract an interface later when the need arises. Modern IDEs make "Extract Interface" a two-second refactoring operation.

**The decision framework:**

| Question | If YES | If NO |
|---|---|---|
| Does this cross an I/O boundary (database, network, file system)? | Apply DIP | Probably skip |
| Will there ever be a second implementation? | Apply DIP | Probably skip |
| Do I need to mock this for testing? | Apply DIP | Probably skip |
| Is this a stable framework API? | Skip DIP | Consider applying |
| Is this a value object or DTO? | Skip DIP | Consider applying |
