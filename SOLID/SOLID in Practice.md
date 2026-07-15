---
tags:
  - solid
  - design-principles
  - refactoring
---

## 🔹 How the 5 Principles Work Together

SOLID is not a checklist of five independent rules. The principles form a reinforcing system: applying one often requires or naturally leads to another, and violating one tends to cascade into violations of the rest.

| Principle | What It Says | Relates to SRP | Relates to OCP | Relates to LSP | Relates to ISP | Relates to DIP |
|-----------|-------------|----------------|----------------|----------------|----------------|----------------|
| [[1 - Single Responsibility Principle\|SRP]] | A class should have only one reason to change | -- | Focused classes are easier to extend without modification | Focused classes are less likely to create broken hierarchies | Narrow responsibilities produce narrow interfaces naturally | Focused classes are easier to abstract behind interfaces |
| [[2 - Open-Closed Principle\|OCP]] | Open for extension, closed for modification | Extending via new classes keeps each class focused | -- | New extensions must honor the base contract or the system breaks | Extension points work best through narrow, cohesive interfaces | Abstractions (DIP) are the mechanism that makes OCP possible |
| [[3 - Liskov Substitution Principle\|LSP]] | Subtypes must be substitutable for their base types | If a base class has too many responsibilities, subtypes are more likely to violate some of them | Violations break OCP because callers need type-checks, reopening closed code | -- | Fat interfaces force implementers to throw `NotSupportedException`, breaking substitutability | Abstractions only work if all implementations honor the contract |
| [[4 - Interface Segregation Principle\|ISP]] | Clients should not depend on methods they do not use | Segregated interfaces mirror single responsibilities | Smaller interfaces are easier to implement for new extensions | Implementers of narrow interfaces are less likely to violate the contract | -- | Narrow interfaces make better abstraction boundaries for DIP |
| [[5 - Dependency Inversion Principle\|DIP]] | Depend on abstractions, not concretions | Abstractions force you to define what a responsibility IS | Abstractions are the seam that lets you extend without modifying | Abstractions define the contract that LSP enforces | DIP works best when the abstractions it depends on are segregated | -- |

### Key synergies

**Fixing OCP often requires DIP.** You cannot make a class open for extension if it is tightly coupled to concrete dependencies. To introduce a strategy pattern (OCP), you first need an abstraction the class depends on (DIP). The two principles are almost always applied together.

**ISP violations lead to LSP violations.** When an interface is too broad, implementers are forced to include methods that do not apply to them. Those methods typically throw `NotImplementedException` or return dummy values, which breaks the substitutability guarantee. Segregating the interface (ISP) eliminates the pressure to create broken implementations (LSP).

**SRP creates the raw material for DIP.** When you extract responsibilities into focused classes, each one becomes a candidate for abstraction. A class with five responsibilities tangled together is almost impossible to hide behind a clean interface. After applying SRP, each extracted class naturally maps to an interface (DIP).

**OCP and LSP are two sides of the same coin.** OCP says you extend the system by adding new types. LSP says those new types must honor the contract. If LSP is violated, OCP breaks because existing code must be modified to handle the misbehaving subtype.

---

## 🔹 Complete Refactoring Example

This section takes a single class that violates all five principles and refactors it step by step. This is the most important part of these notes --- read the violating code carefully before proceeding.

### The violating code

```csharp
// A fat interface that forces implementers to support everything
public interface IInvoiceProcessor
{
    decimal ProcessInvoice(Invoice invoice, string format);
    void SendReminder(Invoice invoice);          // not all processors send reminders
    void ArchiveInvoice(Invoice invoice);         // not all processors archive
    void GenerateMonthlyReport(DateTime month);   // not all processors report
}

public class Invoice
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public string CustomerEmail { get; set; }
    public string CountryCode { get; set; }
    public decimal Amount { get; set; }
    public List<LineItem> LineItems { get; set; } = new();
}

public class LineItem
{
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int Quantity { get; set; }
}

// VIOLATES ALL 5 SOLID PRINCIPLES
public class InvoiceService : IInvoiceProcessor  // ISP: forced to implement methods it doesn't need
{
    public decimal ProcessInvoice(Invoice invoice, string format)
    {
        // --- SRP VIOLATION: validation logic mixed into processing ---
        if (string.IsNullOrEmpty(invoice.CustomerName))
            throw new ArgumentException("Customer name is required.");
        if (invoice.Amount <= 0)
            throw new ArgumentException("Amount must be positive.");
        if (invoice.LineItems == null || invoice.LineItems.Count == 0)
            throw new ArgumentException("Invoice must have at least one line item.");

        // --- OCP VIOLATION: adding a new country means modifying this method ---
        decimal taxRate;
        if (invoice.CountryCode == "US")
            taxRate = 0.07m;
        else if (invoice.CountryCode == "UK")
            taxRate = 0.20m;
        else if (invoice.CountryCode == "DE")
            taxRate = 0.19m;
        else if (invoice.CountryCode == "JP")
            taxRate = 0.10m;
        else
            taxRate = 0.15m;

        decimal tax = invoice.Amount * taxRate;
        decimal total = invoice.Amount + tax;

        // --- DIP VIOLATION: concrete dependency on SqlConnection ---
        using var connection = new SqlConnection("Server=prod;Database=Invoices;");
        connection.Open();
        using var cmd = new SqlCommand(
            "INSERT INTO Invoices (Id, Customer, Amount, Tax, Total) VALUES (@id, @c, @a, @t, @tot)",
            connection);
        cmd.Parameters.AddWithValue("@id", invoice.Id);
        cmd.Parameters.AddWithValue("@c", invoice.CustomerName);
        cmd.Parameters.AddWithValue("@a", invoice.Amount);
        cmd.Parameters.AddWithValue("@t", tax);
        cmd.Parameters.AddWithValue("@tot", total);
        cmd.ExecuteNonQuery();

        // --- DIP VIOLATION: concrete dependency on SmtpClient ---
        using var smtp = new SmtpClient("smtp.company.com");
        var mail = new MailMessage("billing@company.com", invoice.CustomerEmail)
        {
            Subject = $"Invoice #{invoice.Id} Processed",
            Body = $"Dear {invoice.CustomerName}, your invoice total is {total:C}."
        };
        smtp.Send(mail);

        // --- OCP VIOLATION: adding a new format means modifying this method ---
        if (format == "PDF")
        {
            File.WriteAllBytes($"invoice_{invoice.Id}.pdf",
                GeneratePdfBytes(invoice, total));   // imaginary PDF generation
        }
        else if (format == "CSV")
        {
            File.WriteAllText($"invoice_{invoice.Id}.csv",
                $"{invoice.Id},{invoice.CustomerName},{invoice.Amount},{tax},{total}");
        }
        else
        {
            throw new NotSupportedException($"Format '{format}' is not supported.");
        }

        return total;
    }

    private byte[] GeneratePdfBytes(Invoice inv, decimal total) =>
        System.Text.Encoding.UTF8.GetBytes($"PDF:{inv.Id}:{total}"); // stub

    // --- ISP VIOLATION: forced to implement methods this class doesn't need ---
    public void SendReminder(Invoice invoice) =>
        throw new NotImplementedException();   // LSP: caller expects this to work

    public void ArchiveInvoice(Invoice invoice) =>
        throw new NotImplementedException();   // LSP: caller expects this to work

    public void GenerateMonthlyReport(DateTime month) =>
        throw new NotImplementedException();   // LSP: caller expects this to work
}

// --- LSP VIOLATION: subclass changes base class behavior in unexpected ways ---
public class DiscountInvoiceService : InvoiceService
{
    public new decimal ProcessInvoice(Invoice invoice, string format)
    {
        // Silently caps the amount at 1000 before processing ---
        // callers who hold an InvoiceService reference will never see this behavior.
        // This violates the postcondition: total should equal amount + tax,
        // but here the amount is secretly mutated.
        if (invoice.Amount > 1000)
            invoice.Amount = 1000;

        return base.ProcessInvoice(invoice, format);
    }
}
```

The problems in summary:

| Violation | Where | Consequence |
|-----------|-------|-------------|
| SRP | Validation, tax calc, persistence, notification, formatting all in one method | Any change to any responsibility risks breaking the others |
| OCP | `if/else` chains for country tax rates and output formats | Adding a country or format requires modifying `ProcessInvoice` |
| LSP | `DiscountInvoiceService` silently mutates the invoice amount | Callers holding the base type get unexpected behavior |
| ISP | `IInvoiceProcessor` forces `SendReminder`, `ArchiveInvoice`, `GenerateMonthlyReport` | `InvoiceService` throws `NotImplementedException` for three methods |
| DIP | Direct `new SqlConnection` and `new SmtpClient` inside business logic | Cannot unit test, cannot swap databases or email providers |

### Step 1 --- Apply SRP: Extract responsibilities

Each distinct responsibility becomes its own class.

```csharp
public class InvoiceValidator
{
    public void Validate(Invoice invoice)
    {
        if (string.IsNullOrEmpty(invoice.CustomerName))
            throw new ArgumentException("Customer name is required.");
        if (invoice.Amount <= 0)
            throw new ArgumentException("Amount must be positive.");
        if (invoice.LineItems == null || invoice.LineItems.Count == 0)
            throw new ArgumentException("Invoice must have at least one line item.");
    }
}

public class TaxCalculator
{
    public decimal CalculateTax(Invoice invoice)
    {
        decimal taxRate = invoice.CountryCode switch
        {
            "US" => 0.07m,
            "UK" => 0.20m,
            "DE" => 0.19m,
            "JP" => 0.10m,
            _    => 0.15m
        };
        return invoice.Amount * taxRate;
    }
}

public class InvoiceRepository
{
    public void Save(Invoice invoice, decimal tax, decimal total)
    {
        using var connection = new SqlConnection("Server=prod;Database=Invoices;");
        connection.Open();
        using var cmd = new SqlCommand(
            "INSERT INTO Invoices (Id, Customer, Amount, Tax, Total) VALUES (@id, @c, @a, @t, @tot)",
            connection);
        cmd.Parameters.AddWithValue("@id", invoice.Id);
        cmd.Parameters.AddWithValue("@c", invoice.CustomerName);
        cmd.Parameters.AddWithValue("@a", invoice.Amount);
        cmd.Parameters.AddWithValue("@t", tax);
        cmd.Parameters.AddWithValue("@tot", total);
        cmd.ExecuteNonQuery();
    }
}

public class EmailNotificationService
{
    public void SendInvoiceNotification(Invoice invoice, decimal total)
    {
        using var smtp = new SmtpClient("smtp.company.com");
        var mail = new MailMessage("billing@company.com", invoice.CustomerEmail)
        {
            Subject = $"Invoice #{invoice.Id} Processed",
            Body = $"Dear {invoice.CustomerName}, your invoice total is {total:C}."
        };
        smtp.Send(mail);
    }
}

public class PdfInvoiceFormatter
{
    public void Generate(Invoice invoice, decimal total)
    {
        byte[] bytes = System.Text.Encoding.UTF8.GetBytes($"PDF:{invoice.Id}:{total}");
        File.WriteAllBytes($"invoice_{invoice.Id}.pdf", bytes);
    }
}

public class CsvInvoiceFormatter
{
    public void Generate(Invoice invoice, decimal tax, decimal total)
    {
        string content = $"{invoice.Id},{invoice.CustomerName},{invoice.Amount},{tax},{total}";
        File.WriteAllText($"invoice_{invoice.Id}.csv", content);
    }
}
```

SRP is now satisfied: each class has exactly one reason to change. But the classes are still concrete and the tax/format logic still uses conditional branching. The next steps address that.

### Step 2 --- Apply OCP: Make tax and formatting extensible

Replace conditional logic with polymorphism using the Strategy pattern.

```csharp
// --- Tax strategies: add a new country by adding a new class, never modifying existing ones ---

public interface ITaxStrategy
{
    string CountryCode { get; }
    decimal CalculateTax(decimal amount);
}

public class UsTaxStrategy : ITaxStrategy
{
    public string CountryCode => "US";
    public decimal CalculateTax(decimal amount) => amount * 0.07m;
}

public class UkTaxStrategy : ITaxStrategy
{
    public string CountryCode => "UK";
    public decimal CalculateTax(decimal amount) => amount * 0.20m;
}

public class GermanyTaxStrategy : ITaxStrategy
{
    public string CountryCode => "DE";
    public decimal CalculateTax(decimal amount) => amount * 0.19m;
}

public class JapanTaxStrategy : ITaxStrategy
{
    public string CountryCode => "JP";
    public decimal CalculateTax(decimal amount) => amount * 0.10m;
}

public class DefaultTaxStrategy : ITaxStrategy
{
    public string CountryCode => "DEFAULT";
    public decimal CalculateTax(decimal amount) => amount * 0.15m;
}

// Resolver that picks the right strategy --- new countries require zero modification here
public class TaxCalculator
{
    private readonly Dictionary<string, ITaxStrategy> _strategies;
    private readonly ITaxStrategy _defaultStrategy;

    public TaxCalculator(IEnumerable<ITaxStrategy> strategies, ITaxStrategy defaultStrategy)
    {
        _strategies = strategies.ToDictionary(s => s.CountryCode, s => s);
        _defaultStrategy = defaultStrategy;
    }

    public decimal CalculateTax(Invoice invoice)
    {
        var strategy = _strategies.GetValueOrDefault(invoice.CountryCode, _defaultStrategy);
        return strategy.CalculateTax(invoice.Amount);
    }
}

// --- Format strategies: add a new format by adding a new class ---

public interface IInvoiceFormatter
{
    string Format { get; }
    void Generate(Invoice invoice, decimal tax, decimal total);
}

public class PdfInvoiceFormatter : IInvoiceFormatter
{
    public string Format => "PDF";

    public void Generate(Invoice invoice, decimal tax, decimal total)
    {
        byte[] bytes = System.Text.Encoding.UTF8.GetBytes($"PDF:{invoice.Id}:{total}");
        File.WriteAllBytes($"invoice_{invoice.Id}.pdf", bytes);
    }
}

public class CsvInvoiceFormatter : IInvoiceFormatter
{
    public string Format => "CSV";

    public void Generate(Invoice invoice, decimal tax, decimal total)
    {
        string content = $"{invoice.Id},{invoice.CustomerName},{invoice.Amount},{tax},{total}";
        File.WriteAllText($"invoice_{invoice.Id}.csv", content);
    }
}

// Resolver --- adding XML format means adding XmlInvoiceFormatter, nothing else changes
public class InvoiceFormatterResolver
{
    private readonly Dictionary<string, IInvoiceFormatter> _formatters;

    public InvoiceFormatterResolver(IEnumerable<IInvoiceFormatter> formatters)
    {
        _formatters = formatters.ToDictionary(f => f.Format, f => f);
    }

    public IInvoiceFormatter Resolve(string format)
    {
        if (!_formatters.TryGetValue(format, out var formatter))
            throw new NotSupportedException($"Format '{format}' is not supported.");
        return formatter;
    }
}
```

OCP is now satisfied. To add Canada's tax rate, you create `CanadaTaxStrategy` and register it. To add XML output, you create `XmlInvoiceFormatter` and register it. No existing class changes.

### Step 3 --- Apply LSP: Ensure subtypes honor contracts

The original `DiscountInvoiceService` used `new` to hide the base method and silently mutated the invoice amount. This violates the postcondition that the returned total equals `amount + tax` on the original amount.

The fix is to make discounting an explicit, transparent behavior rather than a sneaky override:

```csharp
// Instead of a subclass that secretly mutates state, use a decorator or
// a separate discount strategy that is explicit in the pipeline.

public interface IDiscountStrategy
{
    decimal ApplyDiscount(Invoice invoice);
}

public class NoDiscount : IDiscountStrategy
{
    public decimal ApplyDiscount(Invoice invoice) => invoice.Amount;
}

public class CappedDiscount : IDiscountStrategy
{
    private readonly decimal _cap;

    public CappedDiscount(decimal cap)
    {
        _cap = cap;
    }

    // Returns the effective amount --- does NOT mutate the invoice.
    // The contract is clear: the original invoice is unchanged,
    // and the returned value is the amount to use for tax calculation.
    public decimal ApplyDiscount(Invoice invoice)
    {
        return Math.Min(invoice.Amount, _cap);
    }
}
```

LSP is now satisfied. Every `IDiscountStrategy` implementation returns a decimal without side effects. Callers do not need to know or care which strategy is in play --- they all honor the same contract: input an invoice, output the effective amount, do not mutate state.

This also means the `ITaxStrategy` implementations must honor their contract: given a non-negative amount, return a non-negative tax. No implementation should throw, return negative values, or produce side effects.

### Step 4 --- Apply ISP: Segregate the fat interface

The original `IInvoiceProcessor` forced implementers to support four unrelated operations. Split it into focused interfaces:

```csharp
public interface IInvoiceValidator
{
    void Validate(Invoice invoice);
}

public interface ITaxCalculator
{
    decimal CalculateTax(Invoice invoice);
}

public interface IInvoiceRepository
{
    void Save(Invoice invoice, decimal tax, decimal total);
}

public interface INotificationService
{
    void SendInvoiceNotification(Invoice invoice, decimal total);
}

// IInvoiceFormatter and IDiscountStrategy were already defined above.
```

Each client depends only on the interface it actually uses. The `InvoiceService` orchestrator depends on all five, but each collaborator class implements only one. No class is ever forced to throw `NotImplementedException`.

### Step 5 --- Apply DIP: Depend on abstractions, inject via constructor

Every concrete dependency is now behind an interface. The `InvoiceService` becomes a thin orchestrator:

```csharp
public class InvoiceService
{
    private readonly IInvoiceValidator _validator;
    private readonly IDiscountStrategy _discountStrategy;
    private readonly ITaxCalculator _taxCalculator;
    private readonly IInvoiceRepository _repository;
    private readonly INotificationService _notificationService;
    private readonly InvoiceFormatterResolver _formatterResolver;

    public InvoiceService(
        IInvoiceValidator validator,
        IDiscountStrategy discountStrategy,
        ITaxCalculator taxCalculator,
        IInvoiceRepository repository,
        INotificationService notificationService,
        InvoiceFormatterResolver formatterResolver)
    {
        _validator = validator;
        _discountStrategy = discountStrategy;
        _taxCalculator = taxCalculator;
        _repository = repository;
        _notificationService = notificationService;
        _formatterResolver = formatterResolver;
    }

    public decimal ProcessInvoice(Invoice invoice, string format)
    {
        _validator.Validate(invoice);

        decimal effectiveAmount = _discountStrategy.ApplyDiscount(invoice);

        // Create a working copy with the effective amount for tax calculation
        var taxableInvoice = new Invoice
        {
            Id = invoice.Id,
            CustomerName = invoice.CustomerName,
            CustomerEmail = invoice.CustomerEmail,
            CountryCode = invoice.CountryCode,
            Amount = effectiveAmount,
            LineItems = invoice.LineItems
        };

        decimal tax = _taxCalculator.CalculateTax(taxableInvoice);
        decimal total = effectiveAmount + tax;

        _repository.Save(invoice, tax, total);
        _notificationService.SendInvoiceNotification(invoice, total);

        var formatter = _formatterResolver.Resolve(format);
        formatter.Generate(invoice, tax, total);

        return total;
    }
}
```

DIP is now satisfied. `InvoiceService` depends only on abstractions. Every concrete implementation is injected from outside. The class can be unit tested with fakes, the database can be swapped, the email provider can be swapped --- all without touching this code.

### The final architecture

```
IInvoiceValidator           IDiscountStrategy         ITaxStrategy (multiple)
       |                          |                         |
InvoiceValidator            CappedDiscount             UsTaxStrategy
                            NoDiscount                 UkTaxStrategy
                                                       GermanyTaxStrategy
                                                       JapanTaxStrategy
                                                       DefaultTaxStrategy
                                                            |
ITaxCalculator -------- TaxCalculator (uses ITaxStrategy dictionary)

IInvoiceRepository          INotificationService       IInvoiceFormatter
       |                          |                         |
InvoiceRepository           EmailNotificationService   PdfInvoiceFormatter
                                                       CsvInvoiceFormatter
                                                            |
                                            InvoiceFormatterResolver

                    InvoiceService (orchestrator)
                 depends on all abstractions above
                 injected via constructor
```

Concrete implementations at the bottom. Abstractions at the top. High-level policy (`InvoiceService`) depends only on abstractions. Low-level details (SQL, SMTP, file I/O) implement those abstractions. The dependency arrows point upward toward abstractions --- exactly what DIP prescribes.

---

## 🔹 SOLID Violation Smell Guide

Use this table as a diagnostic tool. When you notice a code smell, check the likely violation column and apply the corresponding fix.

| Smell | Likely Violation | Fix |
|-------|-----------------|-----|
| Giant class (500+ lines) | SRP | Extract classes by responsibility --- each class should have one reason to change |
| Long `if/else` or `switch` statements | OCP | Replace with Strategy pattern or polymorphism --- add behavior by adding classes |
| Type checking with `is`/`as` in business logic | LSP | Redesign the hierarchy so all subtypes are substitutable without runtime checks |
| `NotImplementedException` in interface methods | ISP + LSP | Segregate the interface into smaller ones that each implementer can fully support |
| `new` keyword for dependencies in business logic | DIP | Extract an interface and inject the dependency via constructor |
| Hard to unit test (needs database, network, file system) | DIP | Depend on abstractions; inject test doubles |
| Shotgun surgery (one logical change touches many files) | SRP | Group related logic into one class so a single change stays in one place |
| Divergent change (one class changes for many unrelated reasons) | SRP | Split the class so each resulting class changes for only one reason |
| Feature envy (method uses another class's data more than its own) | SRP | Move the method to the class whose data it primarily uses |
| Parallel inheritance hierarchies (adding a subclass in one hierarchy forces a subclass in another) | OCP / LSP | Replace inheritance with composition; use strategy or visitor patterns |
| God class / Manager class / Service class doing everything | SRP | Decompose into focused collaborators, each with a single responsibility |
| Empty catch blocks or silently swallowed exceptions | LSP | Subtypes and methods should propagate errors according to their contract, not hide them |
| Boolean parameters that change method behavior (`process(invoice, bool sendEmail)`) | OCP / SRP | Split into separate methods or classes; use strategy or decorator instead |
| Refused bequest (subclass inherits methods it does not use or overrides to do nothing) | LSP / ISP | Favor composition over inheritance; extract the unused behavior into a separate interface |
| Concrete class references in constructor parameters | DIP | Introduce an interface; depend on the abstraction |
| Static utility methods with side effects | DIP / SRP | Wrap in an instance class behind an interface so it can be injected and tested |
| Circular dependencies between classes | DIP | Introduce an abstraction that both classes depend on, breaking the cycle |
| Methods that return different types based on input | LSP | Define a consistent return contract; use generics or a common base type |
| Config values hard-coded in class bodies | DIP / OCP | Inject configuration through constructor or options pattern |
| Unit tests require complex setup or real infrastructure | DIP | The class under test depends on concretions; introduce interfaces and inject fakes |

---

## 🔹 SOLID and Design Patterns

Each SOLID principle has design patterns that serve as its primary implementation mechanism. Understanding this mapping helps you pick the right pattern when you identify a violation.

| Principle | Patterns | How It Helps |
|-----------|----------|--------------|
| SRP | **Facade** | Provides a unified interface to a set of focused classes, hiding the fact that responsibilities have been split across multiple collaborators |
| SRP | **Mediator** | Decouples objects that interact in complex ways, ensuring each object has a single responsibility and communicates through the mediator rather than knowing about every collaborator |
| SRP | **Command** | Encapsulates a request as an object, separating "what to do" from "when and how to invoke it" --- each command class has one responsibility |
| OCP | **Strategy** | Encapsulates interchangeable algorithms behind a common interface; new algorithms are added by creating new classes, not modifying existing ones |
| OCP | **Decorator** | Adds behavior to objects dynamically by wrapping them; new behavior is a new decorator class, the original class is untouched |
| OCP | **Template Method** | Defines the skeleton of an algorithm in a base class; subclasses override specific steps without changing the overall structure |
| OCP | **Observer** | Lets objects subscribe to events; new subscribers are added without modifying the publisher |
| OCP | **Factory Method** | Defers object creation to subclasses; new product types are added by creating new factory subclasses |
| LSP | **(Design guideline)** | LSP is not implemented by a specific pattern --- it is a constraint that all patterns using inheritance or polymorphism must satisfy. It governs HOW you use Template Method, Strategy, and any other pattern involving substitution. The "fix" for LSP violations is usually to redesign contracts: tighten preconditions in the base, ensure postconditions hold in all subtypes, and prefer composition over inheritance when substitutability is difficult to guarantee |
| ISP | **Adapter** | Adapts a broad interface to a narrow one that the client actually needs, effectively providing a segregated view of a larger API |
| ISP | **Role Interface** | Defines multiple small interfaces, each representing a role the implementing class can play; clients depend only on the role they use |
| DIP | **Factory / Abstract Factory** | Creates objects through abstractions so the client never references concrete types directly |
| DIP | **DI Container** | Automates the wiring of abstractions to implementations at composition root; the entire application depends on abstractions resolved at startup |
| DIP | **Repository** | Abstracts data access behind an interface; business logic depends on `IRepository`, not on `SqlConnection` or `DbContext` |
| DIP | **Strategy** | (Also serves DIP) The strategy interface is the abstraction that high-level code depends on; concrete strategies are the low-level details |

### Why Strategy appears under both OCP and DIP

Strategy is the single most SOLID-aligned pattern. It makes code open for extension (OCP) by allowing new algorithms without modifying existing code. It simultaneously satisfies DIP because the consuming class depends on the `IStrategy` interface, not on any concrete algorithm. When you find yourself fixing both an OCP and a DIP violation in the same class, the Strategy pattern is almost always the answer.

---

## 🔹 SOLID and Testing

Each SOLID principle contributes to testability in a different way. Together, they make the difference between code that is trivial to test and code that requires integration tests with real databases.

### How each principle enables testing

**SRP makes tests focused.** When a class has one responsibility, its tests cover one thing. You do not need to set up a database, an email server, AND a file system just to test tax calculation. Each test class mirrors a production class with a clear, narrow scope.

**OCP makes tests stable.** When new behavior is added by creating new classes rather than modifying existing ones, existing tests never break. Adding `CanadaTaxStrategy` means writing new tests for that class --- existing `UsTaxStrategy` tests are untouched.

**LSP makes tests reliable.** When all implementations of an interface honor the same contract, you can write contract tests once and run them against every implementation. If a test passes for the base type, it must pass for every subtype. Any subtype that fails reveals an LSP violation.

**ISP makes mocking easier.** When interfaces are narrow, mock setup is minimal. Mocking `IInvoiceValidator` (one method) is trivial. Mocking the original `IInvoiceProcessor` (four methods) requires setting up stubs for methods the test does not even care about.

**DIP enables test doubles entirely.** Without DIP, the class creates its own dependencies with `new`. You cannot intercept `new SqlConnection(...)` in a unit test. DIP replaces the concrete creation with constructor injection, letting you pass in fakes, stubs, or mocks.

### Complete unit test for the refactored InvoiceService

This example uses manual test doubles (fakes) to avoid external dependencies. The same approach works with Moq --- a Moq-based version follows.

```csharp
// --- Fakes for testing ---

public class FakeValidator : IInvoiceValidator
{
    public bool WasCalled { get; private set; }
    public bool ShouldThrow { get; set; }

    public void Validate(Invoice invoice)
    {
        WasCalled = true;
        if (ShouldThrow)
            throw new ArgumentException("Validation failed.");
    }
}

public class FakeDiscountStrategy : IDiscountStrategy
{
    public decimal ApplyDiscount(Invoice invoice) => invoice.Amount; // no discount
}

public class FakeTaxCalculator : ITaxCalculator
{
    public decimal FixedTax { get; set; } = 10m;
    public decimal CalculateTax(Invoice invoice) => FixedTax;
}

public class FakeRepository : IInvoiceRepository
{
    public Invoice SavedInvoice { get; private set; }
    public decimal SavedTax { get; private set; }
    public decimal SavedTotal { get; private set; }

    public void Save(Invoice invoice, decimal tax, decimal total)
    {
        SavedInvoice = invoice;
        SavedTax = tax;
        SavedTotal = total;
    }
}

public class FakeNotificationService : INotificationService
{
    public bool WasCalled { get; private set; }
    public void SendInvoiceNotification(Invoice invoice, decimal total) => WasCalled = true;
}

public class FakeFormatter : IInvoiceFormatter
{
    public string Format => "FAKE";
    public bool WasCalled { get; private set; }

    public void Generate(Invoice invoice, decimal tax, decimal total) => WasCalled = true;
}

// --- The test class ---

[TestClass]
public class InvoiceServiceTests
{
    private FakeValidator _validator;
    private FakeDiscountStrategy _discountStrategy;
    private FakeTaxCalculator _taxCalculator;
    private FakeRepository _repository;
    private FakeNotificationService _notification;
    private FakeFormatter _formatter;
    private InvoiceService _sut; // system under test

    [TestInitialize]
    public void Setup()
    {
        _validator = new FakeValidator();
        _discountStrategy = new FakeDiscountStrategy();
        _taxCalculator = new FakeTaxCalculator { FixedTax = 10m };
        _repository = new FakeRepository();
        _notification = new FakeNotificationService();
        _formatter = new FakeFormatter();

        var resolver = new InvoiceFormatterResolver(new[] { _formatter });

        _sut = new InvoiceService(
            _validator,
            _discountStrategy,
            _taxCalculator,
            _repository,
            _notification,
            resolver);
    }

    private static Invoice CreateValidInvoice() => new Invoice
    {
        Id = 1,
        CustomerName = "Acme Corp",
        CustomerEmail = "billing@acme.com",
        CountryCode = "US",
        Amount = 100m,
        LineItems = new List<LineItem>
        {
            new LineItem { Description = "Widget", Price = 100m, Quantity = 1 }
        }
    };

    [TestMethod]
    public void ProcessInvoice_ValidInvoice_ReturnsTotalWithTax()
    {
        var invoice = CreateValidInvoice();
        _taxCalculator.FixedTax = 7m;

        decimal total = _sut.ProcessInvoice(invoice, "FAKE");

        Assert.AreEqual(107m, total); // 100 + 7
    }

    [TestMethod]
    public void ProcessInvoice_CallsValidator()
    {
        var invoice = CreateValidInvoice();

        _sut.ProcessInvoice(invoice, "FAKE");

        Assert.IsTrue(_validator.WasCalled);
    }

    [TestMethod]
    [ExpectedException(typeof(ArgumentException))]
    public void ProcessInvoice_InvalidInvoice_ThrowsFromValidator()
    {
        _validator.ShouldThrow = true;
        var invoice = CreateValidInvoice();

        _sut.ProcessInvoice(invoice, "FAKE");
    }

    [TestMethod]
    public void ProcessInvoice_SavesInvoiceToRepository()
    {
        var invoice = CreateValidInvoice();
        _taxCalculator.FixedTax = 10m;

        _sut.ProcessInvoice(invoice, "FAKE");

        Assert.AreEqual(invoice.Id, _repository.SavedInvoice.Id);
        Assert.AreEqual(10m, _repository.SavedTax);
        Assert.AreEqual(110m, _repository.SavedTotal);
    }

    [TestMethod]
    public void ProcessInvoice_SendsNotification()
    {
        var invoice = CreateValidInvoice();

        _sut.ProcessInvoice(invoice, "FAKE");

        Assert.IsTrue(_notification.WasCalled);
    }

    [TestMethod]
    public void ProcessInvoice_CallsFormatter()
    {
        var invoice = CreateValidInvoice();

        _sut.ProcessInvoice(invoice, "FAKE");

        Assert.IsTrue(_formatter.WasCalled);
    }

    [TestMethod]
    [ExpectedException(typeof(NotSupportedException))]
    public void ProcessInvoice_UnsupportedFormat_Throws()
    {
        var invoice = CreateValidInvoice();

        _sut.ProcessInvoice(invoice, "XML"); // no XML formatter registered
    }
}
```

### Moq-based version (abbreviated)

```csharp
[TestClass]
public class InvoiceServiceMoqTests
{
    [TestMethod]
    public void ProcessInvoice_ValidInvoice_OrchestratesAllDependencies()
    {
        // Arrange
        var validator = new Mock<IInvoiceValidator>();
        var discount = new Mock<IDiscountStrategy>();
        var taxCalc = new Mock<ITaxCalculator>();
        var repo = new Mock<IInvoiceRepository>();
        var notifier = new Mock<INotificationService>();
        var formatter = new Mock<IInvoiceFormatter>();

        var invoice = new Invoice
        {
            Id = 1, CustomerName = "Acme", CustomerEmail = "a@b.com",
            CountryCode = "US", Amount = 200m,
            LineItems = new List<LineItem>
            {
                new LineItem { Description = "Item", Price = 200m, Quantity = 1 }
            }
        };

        discount.Setup(d => d.ApplyDiscount(invoice)).Returns(200m);
        taxCalc.Setup(t => t.CalculateTax(It.IsAny<Invoice>())).Returns(14m);
        formatter.Setup(f => f.Format).Returns("PDF");

        var resolver = new InvoiceFormatterResolver(new[] { formatter.Object });

        var sut = new InvoiceService(
            validator.Object, discount.Object, taxCalc.Object,
            repo.Object, notifier.Object, resolver);

        // Act
        decimal total = sut.ProcessInvoice(invoice, "PDF");

        // Assert
        Assert.AreEqual(214m, total);
        validator.Verify(v => v.Validate(invoice), Times.Once);
        repo.Verify(r => r.Save(invoice, 14m, 214m), Times.Once);
        notifier.Verify(n => n.SendInvoiceNotification(invoice, 214m), Times.Once);
        formatter.Verify(f => f.Generate(invoice, 14m, 214m), Times.Once);
    }
}
```

Notice how ISP makes mocking trivial: each mock has one or two methods to set up. Compare this to mocking the original `IInvoiceProcessor` with four methods, three of which the test does not care about.

---

## 🔹 SOLID and Clean Architecture

[[Clean Architecture]] is essentially SOLID applied at the architectural level. Each layer of Clean Architecture maps to one or more SOLID principles.

```
+----------------------------------------------------------+
|                    Frameworks & Drivers                    |
|  (ASP.NET, EF Core, SQL Server, SMTP, file system, UI)   |
|                                                           |
|  DIP: This layer implements interfaces defined in inner   |
|       layers. Dependencies point INWARD.                  |
+----------------------------------------------------------+
|                   Interface Adapters                      |
|  (Controllers, Presenters, Gateways, ViewModels)          |
|                                                           |
|  ISP: Each adapter exposes only what its consumer needs.  |
|  DIP: Adapters depend on use case interfaces, not on      |
|       framework types directly.                           |
+----------------------------------------------------------+
|                      Use Cases                            |
|  (Application services, interactors, commands, queries)   |
|                                                           |
|  SRP: Each use case class handles one business operation. |
|  OCP: New use cases are new classes, not modifications    |
|       to existing ones.                                   |
+----------------------------------------------------------+
|                       Entities                            |
|  (Domain models, value objects, domain services)          |
|                                                           |
|  LSP: All entity subtypes must be substitutable.          |
|       Domain invariants hold across the hierarchy.        |
+----------------------------------------------------------+
         Dependencies always point INWARD (DIP)
```

**The Dependency Rule is DIP.** The most important rule of Clean Architecture --- inner layers never reference outer layers --- is the Dependency Inversion Principle at the architectural scale. Use case interfaces are defined in the inner layer. Implementations (repositories, gateways) live in the outer layer and are injected inward.

**Use cases embody SRP and OCP.** Each use case (interactor) handles exactly one business operation (SRP). When a new feature is needed, you add a new use case class rather than modifying existing ones (OCP).

**Entities enforce LSP.** The entity layer defines domain invariants. If `PremiumCustomer` extends `Customer`, it must honor all the invariants that `Customer` established. Any code that operates on `Customer` must work correctly when given a `PremiumCustomer`.

**Interface adapters apply ISP.** Controllers, presenters, and gateways each consume only the specific use case interfaces they need. A `BillingController` depends on `IProcessInvoice`, not on a monolithic `IAllUseCases` interface.

---

## 🔹 When SOLID Is Overkill

SOLID is a set of guidelines for managing complexity in systems that grow and change. Not every piece of code grows or changes. Applying SOLID prematurely adds accidental complexity that costs more than the flexibility it provides.

### When to skip or defer SOLID

**Small scripts and one-off tools.** A 50-line console app that parses a CSV file and writes results to another file does not need interfaces, dependency injection, or a strategy pattern. The overhead of the abstraction exceeds the benefit.

**Prototypes and throwaway code.** If the code will be discarded after validation, invest in speed of writing rather than maintainability. You can always refactor toward SOLID if the prototype becomes a product.

**Stable code that will never change.** A utility class that has not been modified in two years and has no reason to change does not benefit from being wrapped in interfaces. SOLID protects against the cost of change --- if there is no change, there is no cost.

### Over-engineering warning signs

- **More interfaces than classes.** If every class has a corresponding interface and most interfaces have exactly one implementation, you have created abstraction without purpose. Interfaces earn their keep when there are (or will be) multiple implementations, or when they serve as a seam for testing a class with side effects.
- **Single-implementation interfaces everywhere.** `IFooService` with only `FooService` implementing it, across the entire codebase. The interface adds a layer of indirection with no polymorphic benefit. Exception: interfaces for classes that touch infrastructure (database, network, file system) are justified even with one implementation because they enable testing.
- **10 files for a simple feature.** If adding a "hello world" endpoint requires creating an interface, an implementation, a factory, a validator, a DTO, a mapper, a repository, a unit test for each, and a registration in the DI container, the architecture has become the problem it was meant to solve.

### The Rule of Three

Do not abstract on the first occurrence. Do not abstract on the second occurrence. Abstract on the third occurrence. The first two times you see a pattern, you do not yet know what the stable abstraction looks like. By the third time, the pattern is clear enough to extract a clean interface.

```
First time:  Write concrete code. It works. Move on.
Second time: Notice the duplication. Resist the urge. Add a comment noting the similarity.
Third time:  Now you have three examples. Extract the common abstraction.
```

### YAGNI vs SOLID

YAGNI ("You Aren't Gonna Need It") and SOLID are not opponents --- they operate on different time horizons.

- **YAGNI** says: do not build features or abstractions for requirements that do not exist yet.
- **SOLID** says: when you do build something, structure it so that future changes are cheap.

The balance: apply SOLID to code that is actively changing or that you have concrete evidence will change. Apply YAGNI to code where the "future flexibility" is purely speculative.

**The cost of premature abstraction** is real: more files to navigate, more indirection to trace, more cognitive load for the next developer, and --- most insidiously --- the wrong abstraction. An abstraction built on speculation often does not match the actual requirement when it arrives, forcing you to tear it out and rebuild. A concrete implementation is easier to refactor into the right abstraction later than a wrong abstraction is to fix.

### The practical heuristic

Start concrete. When a class grows past ~200 lines, or when a method has multiple `if/else` branches for different cases, or when you cannot write a unit test without a database --- THEN apply the relevant SOLID principle. Not before.

---

## 🔹 Incremental Adoption

You do not need to refactor an entire codebase toward SOLID in one pass. The principles can be adopted incrementally, and some deliver more immediate value than others.

### Recommended order

**1. Start with DIP (biggest immediate impact for testability).** The single most impactful change you can make to a legacy codebase is to start injecting dependencies instead of creating them inline. This immediately enables unit testing, which gives you a safety net for all subsequent refactoring.

Practical first step: take one service class that uses `new SqlConnection(...)` or `new HttpClient()` directly. Extract an interface (`IInvoiceRepository`, `IHttpService`), inject it via the constructor, and write one unit test with a fake. Repeat for the next class.

**2. Then SRP (biggest impact for maintainability).** Once you have dependency injection in place, start splitting classes that have grown too large. Look for classes over 200 lines or classes that change for multiple reasons. Extract responsibilities into focused collaborators and inject them.

**3. ISP and OCP follow naturally.** As you split classes (SRP) and inject dependencies (DIP), you will notice that some interfaces are too broad (ISP) and some switch statements keep growing (OCP). These violations become obvious and easy to fix once the foundation of SRP and DIP is in place.

**4. LSP is more about awareness than active refactoring.** LSP violations are rare if your inheritance hierarchies are shallow (prefer composition). The main action item is to be vigilant when creating subclasses: ensure they honor the base contract. If you find yourself overriding a method to throw `NotSupportedException` or to do nothing, that is a signal to use composition instead.

### Introducing SOLID into a legacy codebase

**Do not rewrite.** The urge to throw everything away and start clean is almost always wrong. The existing code encodes years of bug fixes, edge cases, and domain knowledge that a rewrite will lose.

**Use the Strangler Fig pattern.** Named after a fig tree that grows around its host until the host dies, this pattern lets you gradually replace legacy code with SOLID-compliant code:

1. Identify a specific responsibility in the legacy class (e.g., tax calculation).
2. Write a new, SOLID-compliant class that handles that responsibility (`TaxCalculator` behind `ITaxCalculator`).
3. Modify the legacy class to delegate to the new class instead of doing the work itself.
4. The legacy class shrinks with each extraction. Eventually, it becomes a thin orchestrator or disappears entirely.

```csharp
// Before: legacy class does everything
public class LegacyOrderProcessor
{
    public void Process(Order order)
    {
        // 500 lines of validation, pricing, persistence, notification...
    }
}

// Step 1: Extract pricing into a new SOLID class
public interface IPricingEngine
{
    decimal CalculateTotal(Order order);
}

public class PricingEngine : IPricingEngine
{
    public decimal CalculateTotal(Order order)
    {
        // Clean, tested, SOLID-compliant pricing logic
        return order.Items.Sum(i => i.Price * i.Quantity);
    }
}

// Step 2: Legacy class delegates to the new class
public class LegacyOrderProcessor
{
    private readonly IPricingEngine _pricing;

    public LegacyOrderProcessor(IPricingEngine pricing)
    {
        _pricing = pricing;
    }

    public void Process(Order order)
    {
        // ... validation still inline (will extract next) ...
        decimal total = _pricing.CalculateTotal(order); // delegated
        // ... persistence still inline (will extract next) ...
    }
}
```

Each extraction is a small, safe change. Each extraction makes the legacy class simpler and the new code testable. Over weeks or months, the legacy class is strangled down to nothing.

### Key mindset

SOLID is not a destination. It is a direction. Every step toward SOLID --- even extracting one interface, even splitting one class --- makes the codebase incrementally better. Perfectionism ("we need to make everything SOLID before we ship") is the enemy of progress. Ship incremental improvements. The codebase will improve steadily without the risk and cost of a big rewrite.
