---
tags:
  - solid
  - ocp
  - open-closed
---

## 🔹 Definition

The **Open/Closed Principle (OCP)** is the second principle in [[SOLID Overview|SOLID]], originally stated by **Bertrand Meyer** in 1988:

> **"Software entities (classes, modules, functions) should be open for extension, but closed for modification."**

This means: ==you should be able to add new behavior to a system without changing existing, tested, working code.== New features are added by writing *new* code, not by editing *old* code.

The word "open" means open for extension -- the behavior of the entity can be extended to meet new requirements. The word "closed" means closed for modification -- the existing source code of the entity is not changed. These two requirements seem contradictory, but OCP resolves the tension through **abstraction**: you define stable interfaces or abstract classes that consuming code depends on, and you extend behavior by adding new implementations of those abstractions.

### Why It Matters

- **Reduces risk**: Modifying existing code risks introducing bugs in features that already work. If you never change the `PaymentProcessor`, you can never accidentally break credit card payments while adding PayPal support.
- **Improves stability**: Classes that are closed for modification become increasingly stable over time. They accumulate a track record of working correctly. Every modification resets that track record.
- **Enables parallel development**: Multiple developers can add new features simultaneously by implementing new classes, without creating merge conflicts in existing code.
- **Supports testing**: Existing tests remain valid because existing code does not change. New features bring new tests. You never have the "I changed one line and 47 tests broke" problem.

## 🔹 Two Approaches to OCP

### Meyer's OCP (Inheritance-Based) -- The Original 1988 Formulation

Bertrand Meyer's original formulation used **implementation inheritance** as the extension mechanism. A class is "closed" once its implementation is complete and published. To extend its behavior, you create a subclass that inherits the existing implementation and adds or overrides behavior.

```csharp
// Meyer's approach: extend via inheritance
public class Shape
{
    public virtual double Area()
    {
        return 0; // default
    }
}

public class Circle : Shape
{
    public double Radius { get; }
    public Circle(double radius) => Radius = radius;

    public override double Area() => Math.PI * Radius * Radius;
}

public class Rectangle : Shape
{
    public double Width { get; }
    public double Height { get; }
    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }

    public override double Area() => Width * Height;
}
```

**Limitations of Meyer's approach:**

1. **Fragile base class problem**: Changes to the base class's internal logic can silently break all subclasses. This is a [[3 - Liskov Substitution Principle|LSP]] violation waiting to happen.
2. **Tight coupling**: Subclasses are coupled to the base class's implementation, not just its contract. If the base class's constructor changes, every subclass must change.
3. **Single inheritance limitation**: C# supports only single class inheritance. Once you extend via inheritance, you cannot also extend via another base class. You are locked into a hierarchy.
4. **Deep hierarchies**: Each level of inheritance adds complexity. After 3-4 levels, the behavior of a class becomes very difficult to reason about because it is scattered across multiple files in the hierarchy.

### Polymorphic OCP (Interface-Based) -- The Modern Preferred Approach

Robert C. Martin reformulated OCP to use **polymorphism through interfaces** (or abstract classes) rather than implementation inheritance. Instead of inheriting from a concrete class, you depend on an abstraction (interface or abstract class) and extend by providing new implementations.

```csharp
// Polymorphic OCP: extend via interfaces
public interface IShape
{
    double Area();
}

public class Circle : IShape
{
    public double Radius { get; }
    public Circle(double radius) => Radius = radius;
    public double Area() => Math.PI * Radius * Radius;
}

public class Rectangle : IShape
{
    public double Width { get; }
    public double Height { get; }
    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }
    public double Area() => Width * Height;
}

// Adding a new shape requires ZERO changes to existing code
public class Triangle : IShape
{
    public double Base { get; }
    public double Height { get; }
    public Triangle(double @base, double height)
    {
        Base = @base;
        Height = height;
    }
    public double Area() => 0.5 * Base * Height;
}

// The consuming code never changes
public class AreaCalculator
{
    public double TotalArea(IEnumerable<IShape> shapes)
    {
        return shapes.Sum(s => s.Area());
    }
}
```

`AreaCalculator` is **closed for modification** -- it never needs to change regardless of how many shape types exist. It is **open for extension** -- adding a `Hexagon`, `Ellipse`, or any other shape just means writing a new class that implements `IShape`.

**This is the approach you should use in modern C#.** It avoids the fragile base class problem, supports multiple interface implementation, and creates clean extension points.

> [!tip] Rule of Thumb
> When you hear "Open/Closed Principle" in modern software engineering, assume the polymorphic (interface-based) interpretation unless explicitly told otherwise. This is what your team, your code reviewer, and your interview panel expect.

## 🔹 Violation Example: PaymentProcessor with Giant Switch

Here is a classic OCP violation -- a `PaymentProcessor` that uses a `switch` statement to handle different payment methods:

```csharp
public enum PaymentMethod
{
    CreditCard,
    PayPal,
    BankTransfer
}

public class PaymentProcessor
{
    public PaymentResult Process(Order order, PaymentMethod method)
    {
        switch (method)
        {
            case PaymentMethod.CreditCard:
                // Credit card processing logic
                Console.WriteLine($"Charging credit card for {order.Total:C}");
                var ccClient = new CreditCardGateway("merchant_id_123");
                bool ccSuccess = ccClient.Charge(order.Total, order.CardNumber, order.Cvv);
                return new PaymentResult
                {
                    Success = ccSuccess,
                    TransactionId = Guid.NewGuid().ToString(),
                    Method = "CreditCard"
                };

            case PaymentMethod.PayPal:
                // PayPal processing logic
                Console.WriteLine($"Processing PayPal payment for {order.Total:C}");
                var ppClient = new PayPalClient("client_id", "secret");
                var ppResult = ppClient.CreatePayment(order.Total, order.PayPalEmail);
                return new PaymentResult
                {
                    Success = ppResult.Status == "COMPLETED",
                    TransactionId = ppResult.PaymentId,
                    Method = "PayPal"
                };

            case PaymentMethod.BankTransfer:
                // Bank transfer processing logic
                Console.WriteLine($"Initiating bank transfer for {order.Total:C}");
                var bankClient = new BankApiClient("api_key");
                var bankRef = bankClient.Transfer(order.Total, order.BankAccount, order.RoutingNumber);
                return new PaymentResult
                {
                    Success = bankRef is not null,
                    TransactionId = bankRef ?? "FAILED",
                    Method = "BankTransfer"
                };

            default:
                throw new ArgumentException($"Unsupported payment method: {method}");
        }
    }
}
```

**Why this violates OCP:**

Every time the business adds a new payment method (cryptocurrency, Apple Pay, Google Pay, Buy Now Pay Later), you must:

1. Add a new enum value to `PaymentMethod`.
2. Open `PaymentProcessor.cs` and add a new `case` to the `switch`.
3. The `PaymentProcessor` class **grows with every new payment method**. It is never "done."
4. A developer adding Apple Pay must edit the same file that handles credit cards. A bug introduced in the Apple Pay case could break credit card processing.
5. The class becomes increasingly difficult to test because it has more and more code paths.
6. The `Order` class accumulates properties for every payment method (`CardNumber`, `Cvv`, `PayPalEmail`, `BankAccount`, `CryptoWalletAddress`...) creating a bloated data model.

This pattern is called a **"type code switch"** -- the enum acts as a type discriminator, and the switch dispatches based on it. It is the single most common OCP violation in the wild.

> [!warning] The Growing Switch Smell
> If you have a `switch` or `if-else` chain that **grows every time a new variant is added**, it is almost always an OCP violation. The switch will appear in multiple places throughout the codebase (e.g., in `Process`, in `Validate`, in `FormatReceipt`), and every new variant requires updating all of them. This is the **shotgun surgery** antipattern.

## 🔹 Fixed Example: Strategy Pattern with IDiscountStrategy

Here is a complete refactoring using the Strategy pattern to achieve OCP. Instead of a customer type enum and a switch, each discount strategy is its own class:

**The violation -- discount calculator with switch on customer type:**

```csharp
public enum CustomerType
{
    Regular,
    Premium,
    Employee,
    VIP
}

public class DiscountCalculator
{
    public decimal CalculateDiscount(Order order, CustomerType customerType)
    {
        switch (customerType)
        {
            case CustomerType.Regular:
                return 0m; // no discount

            case CustomerType.Premium:
                return order.Subtotal * 0.10m; // 10% off

            case CustomerType.Employee:
                return order.Subtotal * 0.25m; // 25% off

            case CustomerType.VIP:
                // VIPs get 15% off, plus free shipping value
                decimal vipDiscount = order.Subtotal * 0.15m;
                decimal shippingCredit = order.ShippingCost;
                return vipDiscount + shippingCredit;

            default:
                throw new ArgumentException($"Unknown customer type: {customerType}");
        }
    }
}
```

**The fix -- OCP-compliant strategy pattern:**

```csharp
// The abstraction -- this is the extension point
public interface IDiscountStrategy
{
    decimal CalculateDiscount(Order order);
}

// Each strategy is a self-contained class
public class NoDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(Order order) => 0m;
}

public class PremiumDiscount : IDiscountStrategy
{
    private readonly decimal _percentage;

    public PremiumDiscount(decimal percentage = 0.10m)
    {
        _percentage = percentage;
    }

    public decimal CalculateDiscount(Order order)
    {
        return order.Subtotal * _percentage;
    }
}

public class EmployeeDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(Order order)
    {
        return order.Subtotal * 0.25m;
    }
}

public class VipDiscount : IDiscountStrategy
{
    public decimal CalculateDiscount(Order order)
    {
        decimal percentageDiscount = order.Subtotal * 0.15m;
        decimal shippingCredit = order.ShippingCost;
        return percentageDiscount + shippingCredit;
    }
}

// The consumer depends on the abstraction, not the implementations
public class OrderProcessor
{
    private readonly IDiscountStrategy _discountStrategy;

    public OrderProcessor(IDiscountStrategy discountStrategy)
    {
        _discountStrategy = discountStrategy;
    }

    public decimal CalculateTotal(Order order)
    {
        decimal discount = _discountStrategy.CalculateDiscount(order);
        return order.Subtotal - discount + order.ShippingCost;
    }
}
```

**Adding a new discount type (e.g., seasonal promotion):**

```csharp
// ZERO changes to any existing class. Just add a new one:
public class SeasonalDiscount : IDiscountStrategy
{
    private readonly decimal _percentage;
    private readonly DateOnly _startDate;
    private readonly DateOnly _endDate;

    public SeasonalDiscount(decimal percentage, DateOnly startDate, DateOnly endDate)
    {
        _percentage = percentage;
        _startDate = startDate;
        _endDate = endDate;
    }

    public decimal CalculateDiscount(Order order)
    {
        var today = DateOnly.FromDateTime(DateTime.UtcNow);
        if (today >= _startDate && today <= _endDate)
            return order.Subtotal * _percentage;

        return 0m;
    }
}

// Register in DI:
builder.Services.AddScoped<IDiscountStrategy>(sp =>
    new SeasonalDiscount(0.20m,
        new DateOnly(2026, 11, 25),
        new DateOnly(2026, 12, 1)));
```

`OrderProcessor` never changed. `PremiumDiscount` never changed. `EmployeeDiscount` never changed. The system was extended purely through addition.

## 🔹 More Examples

### Shape Area Calculator (Classic Example)

**Violation:**

```csharp
public class AreaCalculator
{
    public double CalculateTotalArea(object[] shapes)
    {
        double totalArea = 0;

        foreach (var shape in shapes)
        {
            if (shape is Circle circle)
            {
                totalArea += Math.PI * circle.Radius * circle.Radius;
            }
            else if (shape is Rectangle rectangle)
            {
                totalArea += rectangle.Width * rectangle.Height;
            }
            else if (shape is Triangle triangle)
            {
                totalArea += 0.5 * triangle.Base * triangle.Height;
            }
            // Every new shape type requires adding another else-if here
        }

        return totalArea;
    }
}
```

**Fixed:**

```csharp
public interface IShape
{
    double Area();
}

public class Circle : IShape
{
    public double Radius { get; }
    public Circle(double radius) => Radius = radius;
    public double Area() => Math.PI * Radius * Radius;
}

public class Rectangle : IShape
{
    public double Width { get; }
    public double Height { get; }
    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }
    public double Area() => Width * Height;
}

public class Triangle : IShape
{
    public double Base { get; }
    public double Height { get; }
    public Triangle(double @base, double height)
    {
        Base = @base;
        Height = height;
    }
    public double Area() => 0.5 * Base * Height;
}

// This class is CLOSED for modification, OPEN for extension
public class AreaCalculator
{
    public double CalculateTotalArea(IEnumerable<IShape> shapes)
    {
        return shapes.Sum(s => s.Area());
    }
}

// Adding a Hexagon? Just write a new class. AreaCalculator never changes.
public class RegularHexagon : IShape
{
    public double SideLength { get; }
    public RegularHexagon(double sideLength) => SideLength = sideLength;
    public double Area() => (3 * Math.Sqrt(3) / 2) * SideLength * SideLength;
}
```

### Notification System (Email/SMS/Push)

**Violation:**

```csharp
public class NotificationService
{
    private readonly SmtpClient _smtp;
    private readonly TwilioClient _twilio;

    public NotificationService(SmtpClient smtp, TwilioClient twilio)
    {
        _smtp = smtp;
        _twilio = twilio;
    }

    public void Send(string userId, string message, string channel)
    {
        switch (channel.ToLower())
        {
            case "email":
                var user = GetUser(userId);
                _smtp.Send(new MailMessage("noreply@app.com", user.Email, "Notification", message));
                break;

            case "sms":
                var phone = GetUser(userId).Phone;
                _twilio.SendSms("+1555000000", phone, message);
                break;

            // Adding push notifications requires modifying this class:
            // case "push":
            //     var pushToken = GetUser(userId).PushToken;
            //     _firebaseClient.Send(pushToken, message);
            //     break;

            default:
                throw new ArgumentException($"Unknown channel: {channel}");
        }
    }

    private User GetUser(string userId) { /* ... */ }
}
```

**Fixed:**

```csharp
// The abstraction
public interface INotificationChannel
{
    Task SendAsync(string userId, string message);
}

// Each channel is a self-contained class
public class EmailNotificationChannel : INotificationChannel
{
    private readonly SmtpClient _smtp;
    private readonly IUserRepository _users;

    public EmailNotificationChannel(SmtpClient smtp, IUserRepository users)
    {
        _smtp = smtp;
        _users = users;
    }

    public async Task SendAsync(string userId, string message)
    {
        var user = await _users.GetByIdAsync(userId);
        if (user?.Email is null) return;

        var mail = new MailMessage("noreply@app.com", user.Email, "Notification", message);
        await _smtp.SendMailAsync(mail);
    }
}

public class SmsNotificationChannel : INotificationChannel
{
    private readonly TwilioClient _twilio;
    private readonly IUserRepository _users;

    public SmsNotificationChannel(TwilioClient twilio, IUserRepository users)
    {
        _twilio = twilio;
        _users = users;
    }

    public async Task SendAsync(string userId, string message)
    {
        var user = await _users.GetByIdAsync(userId);
        if (user?.Phone is null) return;

        await _twilio.SendSmsAsync("+1555000000", user.Phone, message);
    }
}

// Adding push notifications? Just add a new class:
public class PushNotificationChannel : INotificationChannel
{
    private readonly FirebaseClient _firebase;
    private readonly IUserRepository _users;

    public PushNotificationChannel(FirebaseClient firebase, IUserRepository users)
    {
        _firebase = firebase;
        _users = users;
    }

    public async Task SendAsync(string userId, string message)
    {
        var user = await _users.GetByIdAsync(userId);
        if (user?.PushToken is null) return;

        await _firebase.SendAsync(user.PushToken, new PushPayload { Body = message });
    }
}

// The orchestrator sends to all registered channels
public class NotificationService
{
    private readonly IEnumerable<INotificationChannel> _channels;

    public NotificationService(IEnumerable<INotificationChannel> channels)
    {
        _channels = channels;
    }

    public async Task NotifyAsync(string userId, string message)
    {
        // Send to ALL registered channels in parallel
        var tasks = _channels.Select(c => c.SendAsync(userId, message));
        await Task.WhenAll(tasks);
    }
}

// DI registration -- adding a channel is a one-line config change
builder.Services.AddScoped<INotificationChannel, EmailNotificationChannel>();
builder.Services.AddScoped<INotificationChannel, SmsNotificationChannel>();
builder.Services.AddScoped<INotificationChannel, PushNotificationChannel>();
builder.Services.AddScoped<NotificationService>();
```

`NotificationService` is completely closed for modification. It does not know or care what channels exist. Adding Slack, Teams, Discord, or carrier pigeon notification channels requires zero changes to existing code.

### ASP.NET Core Middleware Pipeline

ASP.NET Core's middleware pipeline is one of the best real-world examples of OCP in the .NET ecosystem. The pipeline itself (`IApplicationBuilder`) is closed for modification. You extend it by adding new middleware -- and you never modify the pipeline infrastructure itself.

```csharp
// Each middleware is a self-contained component
public class RequestTimingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestTimingMiddleware> _logger;

    public RequestTimingMiddleware(RequestDelegate next, ILogger<RequestTimingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var stopwatch = Stopwatch.StartNew();

        await _next(context); // call the next middleware in the pipeline

        stopwatch.Stop();
        _logger.LogInformation(
            "Request {Method} {Path} completed in {Elapsed}ms",
            context.Request.Method,
            context.Request.Path,
            stopwatch.ElapsedMilliseconds);
    }
}

// Registering middleware extends the pipeline without modifying it
var app = builder.Build();

app.UseMiddleware<RequestTimingMiddleware>(); // your custom middleware
app.UseAuthentication();                      // built-in middleware
app.UseAuthorization();                       // built-in middleware
app.MapControllers();

app.Run();
```

The key OCP insight: Microsoft shipped `UseAuthentication()` and `UseAuthorization()`. You shipped `RequestTimingMiddleware`. Neither needed to modify the other's code. The pipeline architecture is the extension point -- it is open for extension (add any middleware) and closed for modification (the pipeline infrastructure never changes).

## 🔹 Patterns That Enable OCP

Several design patterns exist specifically to enable OCP. Each provides a different mechanism for extending behavior without modifying existing code.

| Pattern | How It Enables OCP | When to Use | C# Example |
|---------|-------------------|-------------|------------|
| **Strategy** | Encapsulates interchangeable algorithms behind an interface | Multiple ways to perform the same operation (sorting, discounts, validation) | `IDiscountStrategy` with `PremiumDiscount`, `SeasonalDiscount` |
| **Template Method** | Base class defines the skeleton; subclasses override specific steps | Fixed algorithm structure with variable steps | `abstract ReportGenerator` with `GenerateHeader()`, `GenerateBody()` |
| **Decorator** | Wraps an existing object to add behavior without changing it | Adding cross-cutting concerns (logging, caching, retry) | `CachingRepository : IRepository` wrapping `SqlRepository` |
| **Observer** | Notifies registered listeners when events occur | Decoupling event producers from consumers | `event EventHandler<OrderPlaced>` with multiple subscribers |
| **Factory + DI** | Creates objects through an abstraction, resolved at runtime | Object creation varies by context or configuration | `IPaymentGatewayFactory` returning different gateways per config |

### Strategy Pattern

Already demonstrated above with `IDiscountStrategy`. The core idea: define an interface for an algorithm, implement each variant as a class, and inject the appropriate one.

```csharp
public interface ISortingStrategy<T>
{
    IEnumerable<T> Sort(IEnumerable<T> items);
}

public class QuickSort<T> : ISortingStrategy<T> where T : IComparable<T>
{
    public IEnumerable<T> Sort(IEnumerable<T> items) => items.OrderBy(x => x);
}

public class BubbleSort<T> : ISortingStrategy<T> where T : IComparable<T>
{
    public IEnumerable<T> Sort(IEnumerable<T> items)
    {
        var list = items.ToList();
        for (int i = 0; i < list.Count - 1; i++)
            for (int j = 0; j < list.Count - i - 1; j++)
                if (list[j].CompareTo(list[j + 1]) > 0)
                    (list[j], list[j + 1]) = (list[j + 1], list[j]);
        return list;
    }
}
```

### Template Method Pattern

The base class defines the overall algorithm structure with `abstract` or `virtual` methods for the parts that vary:

```csharp
public abstract class DataExporter
{
    // Template method -- defines the algorithm skeleton
    // This method is CLOSED for modification
    public void Export(IEnumerable<DataRecord> records)
    {
        var header = FormatHeader();
        var body = FormatBody(records);
        var footer = FormatFooter(records);
        var output = Combine(header, body, footer);
        WriteOutput(output);
    }

    // These steps are OPEN for extension via subclasses
    protected abstract string FormatHeader();
    protected abstract string FormatBody(IEnumerable<DataRecord> records);
    protected virtual string FormatFooter(IEnumerable<DataRecord> records)
        => $"Total records: {records.Count()}";
    protected abstract void WriteOutput(string content);

    private string Combine(string header, string body, string footer)
        => $"{header}\n{body}\n{footer}";
}

public class CsvExporter : DataExporter
{
    private readonly string _outputPath;
    public CsvExporter(string outputPath) => _outputPath = outputPath;

    protected override string FormatHeader() => "Id,Name,Value";

    protected override string FormatBody(IEnumerable<DataRecord> records)
        => string.Join("\n", records.Select(r => $"{r.Id},{r.Name},{r.Value}"));

    protected override void WriteOutput(string content)
        => File.WriteAllText(_outputPath, content);
}

public class JsonExporter : DataExporter
{
    private readonly Stream _outputStream;
    public JsonExporter(Stream outputStream) => _outputStream = outputStream;

    protected override string FormatHeader() => "[";

    protected override string FormatBody(IEnumerable<DataRecord> records)
        => string.Join(",\n", records.Select(r =>
            JsonSerializer.Serialize(r)));

    protected override string FormatFooter(IEnumerable<DataRecord> records) => "]";

    protected override void WriteOutput(string content)
    {
        using var writer = new StreamWriter(_outputStream, leaveOpen: true);
        writer.Write(content);
    }
}
```

> [!warning] Template Method Caveat
> Template Method uses inheritance, which creates coupling between base and derived classes. Prefer Strategy (composition-based) when possible. Use Template Method when the algorithm skeleton is genuinely fixed and only specific steps vary.

### Decorator Pattern

Wraps an existing implementation to add behavior, without modifying the wrapped class:

```csharp
public interface IOrderRepository
{
    Order? GetById(int id);
    void Save(Order order);
}

// The real implementation
public class SqlOrderRepository : IOrderRepository
{
    private readonly DbContext _db;
    public SqlOrderRepository(DbContext db) => _db = db;

    public Order? GetById(int id) => _db.Orders.Find(id);
    public void Save(Order order)
    {
        _db.Orders.Add(order);
        _db.SaveChanges();
    }
}

// Decorator: adds caching WITHOUT modifying SqlOrderRepository
public class CachingOrderRepository : IOrderRepository
{
    private readonly IOrderRepository _inner;
    private readonly IMemoryCache _cache;

    public CachingOrderRepository(IOrderRepository inner, IMemoryCache cache)
    {
        _inner = inner;
        _cache = cache;
    }

    public Order? GetById(int id)
    {
        string cacheKey = $"order:{id}";
        if (_cache.TryGetValue(cacheKey, out Order? cached))
            return cached;

        var order = _inner.GetById(id);
        if (order is not null)
            _cache.Set(cacheKey, order, TimeSpan.FromMinutes(5));

        return order;
    }

    public void Save(Order order)
    {
        _inner.Save(order);
        _cache.Remove($"order:{order.Id}"); // invalidate cache
    }
}

// Decorator: adds logging WITHOUT modifying either of the above
public class LoggingOrderRepository : IOrderRepository
{
    private readonly IOrderRepository _inner;
    private readonly ILogger<LoggingOrderRepository> _logger;

    public LoggingOrderRepository(IOrderRepository inner, ILogger<LoggingOrderRepository> logger)
    {
        _inner = inner;
        _logger = logger;
    }

    public Order? GetById(int id)
    {
        _logger.LogDebug("Getting order {OrderId}", id);
        var order = _inner.GetById(id);
        _logger.LogDebug("Order {OrderId} {Result}", id, order is not null ? "found" : "not found");
        return order;
    }

    public void Save(Order order)
    {
        _logger.LogInformation("Saving order {OrderId}", order.Id);
        _inner.Save(order);
        _logger.LogInformation("Order {OrderId} saved successfully", order.Id);
    }
}

// DI registration -- compose the decorators
builder.Services.AddScoped<SqlOrderRepository>();
builder.Services.AddScoped<IOrderRepository>(sp =>
    new LoggingOrderRepository(
        new CachingOrderRepository(
            sp.GetRequiredService<SqlOrderRepository>(),
            sp.GetRequiredService<IMemoryCache>()),
        sp.GetRequiredService<ILogger<LoggingOrderRepository>>()));
```

The Decorator pattern is OCP in its purest form: you **never** modify `SqlOrderRepository`. You wrap it with new behaviors. Each decorator is independent and composable. You can add retry logic, circuit-breaking, rate-limiting, or auditing -- all without touching the original class.

### Observer Pattern

Decouples the event source from the handlers. Adding a new handler never modifies the source:

```csharp
// The event -- define what happened
public record OrderPlacedEvent(int OrderId, string CustomerEmail, decimal Total);

// Handler interface
public interface IOrderPlacedHandler
{
    Task HandleAsync(OrderPlacedEvent @event);
}

// Each handler is independent
public class SendConfirmationEmailHandler : IOrderPlacedHandler
{
    private readonly IEmailService _email;

    public SendConfirmationEmailHandler(IEmailService email) => _email = email;

    public async Task HandleAsync(OrderPlacedEvent @event)
    {
        await _email.SendAsync(@event.CustomerEmail,
            $"Order #{@event.OrderId} confirmed! Total: {@event.Total:C}");
    }
}

public class UpdateInventoryHandler : IOrderPlacedHandler
{
    private readonly IInventoryService _inventory;

    public UpdateInventoryHandler(IInventoryService inventory) => _inventory = inventory;

    public async Task HandleAsync(OrderPlacedEvent @event)
    {
        await _inventory.DeductStockForOrderAsync(@event.OrderId);
    }
}

// Adding a new handler never changes existing code:
public class NotifyWarehouseHandler : IOrderPlacedHandler
{
    private readonly IWarehouseClient _warehouse;

    public NotifyWarehouseHandler(IWarehouseClient warehouse) => _warehouse = warehouse;

    public async Task HandleAsync(OrderPlacedEvent @event)
    {
        await _warehouse.QueueForPickingAsync(@event.OrderId);
    }
}

// The event dispatcher -- closed for modification
public class OrderEventDispatcher
{
    private readonly IEnumerable<IOrderPlacedHandler> _handlers;

    public OrderEventDispatcher(IEnumerable<IOrderPlacedHandler> handlers)
    {
        _handlers = handlers;
    }

    public async Task DispatchAsync(OrderPlacedEvent @event)
    {
        foreach (var handler in _handlers)
        {
            await handler.HandleAsync(@event);
        }
    }
}

// DI registration -- adding a handler is a one-line change
builder.Services.AddScoped<IOrderPlacedHandler, SendConfirmationEmailHandler>();
builder.Services.AddScoped<IOrderPlacedHandler, UpdateInventoryHandler>();
builder.Services.AddScoped<IOrderPlacedHandler, NotifyWarehouseHandler>();
builder.Services.AddScoped<OrderEventDispatcher>();
```

## 🔹 OCP in the .NET Ecosystem

OCP is deeply embedded in how the .NET framework and ecosystem are designed. Understanding these patterns helps you recognize OCP in action.

### ASP.NET Core Middleware

As shown above, the middleware pipeline is the canonical .NET OCP example. The framework defines `RequestDelegate` and `IMiddleware`. You extend the pipeline by adding middleware. The pipeline infrastructure never changes.

### ASP.NET Core Dependency Injection

The `IServiceCollection` / `IServiceProvider` system is OCP-compliant. The DI container is closed for modification (you never edit its source). You extend it by registering new services:

```csharp
builder.Services.AddScoped<IPaymentGateway, StripePaymentGateway>();
// Later, want to add PayPal? Just register another:
builder.Services.AddScoped<IPaymentGateway, PayPalPaymentGateway>();
```

### Entity Framework Core Providers

EF Core's database provider model follows OCP. The core EF framework is closed for modification. Each database is supported by a provider package:

```csharp
// Extend EF Core by adding a provider -- no EF source code changes
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(connectionString));
// or
    options.UseNpgsql(connectionString));   // PostgreSQL
// or
    options.UseSqlite(connectionString));   // SQLite
```

Microsoft ships SQL Server support. Npgsql ships PostgreSQL support. Both extend EF Core without modifying it.

### Logging Providers

The `ILoggerProvider` system (shown in detail in [[4 - Interface Segregation Principle|ISP notes]]) follows OCP. The logging framework is closed. Providers extend it:

```csharp
builder.Logging.AddConsole();       // built-in
builder.Logging.AddDebug();         // built-in
builder.Logging.AddSerilog();       // third-party, extends without modifying
builder.Logging.AddApplicationInsights(); // third-party
```

### Configuration Providers

`IConfigurationProvider` lets you extend the configuration system without modifying it:

```csharp
builder.Configuration
    .AddJsonFile("appsettings.json")      // built-in
    .AddEnvironmentVariables()             // built-in
    .AddAzureKeyVault(vaultUri, credential) // third-party extension
    .AddConsul("config/myapp");            // third-party extension
```

### IOptions Pattern

The options pattern (`IOptions<T>`, `IOptionsMonitor<T>`) uses OCP to allow post-registration configuration:

```csharp
// Extension: configure options without modifying the service that uses them
builder.Services.Configure<SmtpOptions>(builder.Configuration.GetSection("Smtp"));
builder.Services.PostConfigure<SmtpOptions>(options =>
{
    if (string.IsNullOrEmpty(options.FromAddress))
        options.FromAddress = "noreply@default.com";
});
```

## 🔹 Common Mistakes

### Over-Abstracting (Speculative Generality)

The most common mistake with OCP is creating extension points that are never used. Every interface, every abstract class, every strategy pattern adds complexity. If there is only one implementation and no foreseeable second one, the abstraction is premature.

```csharp
// Over-abstracted: IUserRepository, IProductRepository, IOrderRepository,
// IInvoiceRepository, IPaymentRepository -- each with a single implementation
// and no realistic second implementation in sight

// Sometimes a concrete class is fine:
public class UserRepository
{
    private readonly DbContext _db;
    public UserRepository(DbContext db) => _db = db;

    public User? GetById(int id) => _db.Users.Find(id);
    // ...
}
```

> [!tip] When to Abstract
> Create the interface when you **actually need** the second implementation. The first implementation can be a concrete class. When the need arises, "Extract Interface" is a one-keystroke refactoring in Visual Studio or Rider. See [[SOLID Overview|YAGNI discussion in the overview]].

### Confusing OCP with "Never Change Anything"

OCP does not mean you never modify existing code. It means you **design for** extension so that the *most common types of change* (adding a new payment method, a new report format, a new notification channel) do not require modification.

Bug fixes, refactoring, and performance improvements are all valid reasons to modify existing code. OCP is about **new feature additions**, not about ossifying code.

### Applying OCP to Unstable Code

OCP is most valuable for code that is **stable and widely depended upon**. Applying OCP to code that is still being actively designed and iterated on adds overhead without benefit. During early development, keep things concrete and simple. Apply OCP when the code has stabilized and new requirements start arriving as variations on established patterns.

### Ignoring the "Closed" Part

Some developers focus only on "open for extension" and forget "closed for modification." They create extensible systems but then modify the base classes anyway when new requirements arrive. This defeats the purpose -- if you are modifying the base every time, the extension point is not well-designed.

If you find yourself modifying the base class or interface frequently, the abstraction boundary is in the wrong place. Step back and redesign the extension point.

## 🔹 Key Insight: Identify the Most Likely Axes of Change

OCP's practical power comes from **correctly predicting which dimensions of the system will change**. You cannot make everything extensible -- that would be architecture astronaut syndrome. You must choose where to invest in extension points.

The key question is: **"What is most likely to change, and in what way?"**

| Likely Change | Extension Point | Pattern |
|--------------|----------------|---------|
| New payment methods | `IPaymentGateway` | Strategy |
| New report formats | `IReportExporter` | Strategy / Template Method |
| New notification channels | `INotificationChannel` | Strategy + DI collection |
| New business rules | `IRule<T>` / `ISpecification<T>` | Strategy / Specification |
| New cross-cutting concerns | Middleware / Decorators | Decorator / Pipeline |
| New data sources | `IRepository<T>` / `IDataProvider` | Strategy |

**How to identify axes of change:**

1. **Talk to stakeholders.** Product managers know what features are coming. If they say "we might add Apple Pay next quarter," that is an axis of change worth designing for.
2. **Look at the git log.** Files that change frequently are likely on an active axis of change. If `PaymentProcessor.cs` has been modified in 8 of the last 10 sprints, it needs an extension point.
3. **Look for switch statements and type checks.** `switch (paymentType)`, `if (customer is VipCustomer)`, `shape.GetType()` -- these are code smells that indicate a missing abstraction.
4. **Apply the "Rule of Two."** When you need the second variant, refactor to introduce the extension point. Not before, not after.

**What to leave concrete:**

- Code that has not changed in months and has no foreseeable reason to change.
- Code where the abstraction would add significant complexity for minimal benefit.
- Performance-critical hot paths where virtual dispatch or interface indirection has measurable cost.
- Internal implementation details that are not exposed to consumers.

OCP is a strategic principle, not a tactical one. Apply it where it provides the most leverage -- at the boundaries where new requirements most frequently arrive -- and leave the rest concrete and simple. As Robert C. Martin says: *"In many ways, the OCP is at the heart of object-oriented design. If you could redesign a system so that it conformed perfectly to OCP, you would have a system that was infinitely extensible and never needed to be modified."* That is the ideal -- the art is in choosing where to invest in approaching it.
