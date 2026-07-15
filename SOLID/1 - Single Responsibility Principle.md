---
tags:
  - solid
  - srp
  - single-responsibility
---

## 🔹 Definition

The **Single Responsibility Principle (SRP)** is the first principle in [[SOLID Overview|SOLID]], stated by Robert C. Martin:

> **"A class should have one, and only one, reason to change."**

The key phrase is *"reason to change."* Martin later clarified this in *Clean Architecture* (2017) with a more precise formulation:

> **"A module should be responsible to one, and only one, actor."**

An **actor** is a group of people (or a single person) who would request a change. A class that serves multiple actors has multiple reasons to change, and those changes will interfere with each other. SRP says: separate the code so that each class serves exactly one actor.

> [!warning] Common Misconception: "Do One Thing"
> SRP does **not** mean "a class should do only one thing." That interpretation leads to nano-classes with a single method each, creating an explosion of tiny classes wired together with excessive ceremony.
>
> SRP means "a class should have only one *reason to change*" -- one actor whose requirements drive modifications to that class. A class can have multiple methods and do substantial work, as long as all that work serves the same actor and changes for the same reason.
>
> The distinction is subtle but critical. A `ShippingCostCalculator` might have five methods (calculate base cost, apply weight surcharge, apply distance surcharge, apply expedited fee, compute total). That is fine -- all five methods serve the same actor (the shipping department) and change for the same reason (shipping pricing rules change). SRP is satisfied.

## 🔹 The "Reason to Change" = One Actor

To understand SRP, you must understand what "reason to change" actually means. It does **not** mean "the class might need a bug fix" or "the class might need performance optimization." Those are universal reasons that apply to every class. A "reason to change" in SRP terms is a **business-driven** reason -- a stakeholder, department, or policy whose requirements evolve independently.

Consider an `Employee` class:

```csharp
public class Employee
{
    public string Name { get; set; }
    public decimal Salary { get; set; }

    // Actor 1: CFO / Finance department
    public decimal CalculatePay()
    {
        // Pay calculation logic including taxes, deductions, overtime
        return Salary * 1.0m; // simplified
    }

    // Actor 2: CTO / Engineering department
    public int CalculateHoursWorked()
    {
        // Time tracking logic, sprint hours, on-call hours
        return 40; // simplified
    }

    // Actor 3: COO / Operations department
    public void SaveToDatabase()
    {
        // Persistence logic
        Console.WriteLine($"Saving {Name} to database.");
    }
}
```

This class serves **three actors**:
1. The **CFO/Finance** team owns pay calculation rules. When tax laws change or the bonus structure is revised, `CalculatePay` must change.
2. The **CTO/Engineering** team owns time tracking rules. When sprint cadences change or on-call tracking is revised, `CalculateHoursWorked` must change.
3. The **COO/Operations** team (or DBA) owns the persistence mechanism. When the database schema changes or you migrate from SQL Server to PostgreSQL, `SaveToDatabase` must change.

These three actors operate independently. A change requested by Finance should not risk breaking time tracking logic. But because they share a class, a developer modifying `CalculatePay` might accidentally break `CalculateHoursWorked` -- especially if they share private helper methods. The three concerns are **coupled by proximity**, not by necessity.

### The Merge Conflict Dimension

There is also a practical, team-workflow reason. If Finance requests a pay calculation change and Engineering requests a time tracking change in the same sprint, two developers will edit the same file. Merge conflicts are guaranteed. The resolution requires both developers to understand each other's changes -- even though the changes are completely unrelated.

With SRP, the pay logic is in `PayCalculator`, the time logic is in `TimeTracker`, and the persistence logic is in `EmployeeRepository`. The two developers never touch the same file.

## 🔹 Why It Matters

### Changes Are Localized

When each class has a single reason to change, modifications are **contained**. Changing pay calculation logic affects only `PayCalculator`. There is no risk of accidentally breaking time tracking, email sending, or database persistence. The blast radius of every change is minimized.

This compounds over time. In a 10-year-old codebase with thousands of classes, the difference between "change one file" and "change five files and hope nothing else breaks" is the difference between confidence and fear.

### Testing Is Straightforward

A class with one responsibility has a small surface area to test. All test cases relate to the same concern. Setup is simple because the class has few (or no) dependencies unrelated to its purpose.

```csharp
// Easy to test: focused class, clear inputs and outputs
public class ShippingCostCalculator
{
    private readonly decimal _baseRate;
    private readonly decimal _weightMultiplier;

    public ShippingCostCalculator(decimal baseRate, decimal weightMultiplier)
    {
        _baseRate = baseRate;
        _weightMultiplier = weightMultiplier;
    }

    public decimal Calculate(decimal weightKg, decimal distanceKm)
    {
        return _baseRate + (weightKg * _weightMultiplier) + (distanceKm * 0.01m);
    }
}

// Tests are simple and obvious
[Fact]
public void Calculate_WithWeightAndDistance_ReturnsCorrectCost()
{
    var calculator = new ShippingCostCalculator(baseRate: 5.00m, weightMultiplier: 0.50m);

    decimal cost = calculator.Calculate(weightKg: 10, distanceKm: 500);

    Assert.Equal(15.00m, cost); // 5.00 + (10 * 0.50) + (500 * 0.01)
}
```

Compare this to testing a God class that handles shipping, inventory, notifications, and logging -- you would need to set up mocks for email servers, databases, and message queues just to test a shipping calculation.

### Readability and Navigation

Class names become meaningful. When you see `InvoicePdfGenerator`, you know exactly what it does and what it does *not* do. You do not need to read 2000 lines to find the PDF generation logic buried between email formatting and database queries.

New team members can navigate by intent: "Where is the tax calculation?" leads to `TaxCalculator`. "Where do we send confirmation emails?" leads to `OrderConfirmationEmailSender`. The codebase becomes self-documenting through its class structure.

### Reusability

A focused class is a reusable class. An `EmailSender` that only sends emails can be used by the order system, the support ticket system, the user registration system, and the password reset flow. A `UserService` that sends emails *and* validates users *and* persists to a database cannot be reused by anyone -- they would inherit all three responsibilities even if they only need one.

## 🔹 How to Identify SRP Violations

### 1. Too Many Dependencies

If a class constructor takes 6+ dependencies, it is almost certainly doing too much. Each dependency represents a concern the class is involved in. A class that needs an `IOrderRepository`, `IEmailSender`, `IPaymentGateway`, `IInventoryService`, `ILogger`, and `ITaxCalculator` is orchestrating too many concerns directly.

```csharp
// Red flag: too many dependencies
public class OrderService
{
    private readonly IOrderRepository _repo;
    private readonly IEmailSender _email;
    private readonly IPaymentGateway _payment;
    private readonly IInventoryService _inventory;
    private readonly ILogger _logger;
    private readonly ITaxCalculator _tax;
    private readonly IDiscountEngine _discount;
    private readonly IFraudDetector _fraud;

    public OrderService(
        IOrderRepository repo,
        IEmailSender email,
        IPaymentGateway payment,
        IInventoryService inventory,
        ILogger logger,
        ITaxCalculator tax,
        IDiscountEngine discount,
        IFraudDetector fraud)
    {
        _repo = repo;
        _email = email;
        _payment = payment;
        _inventory = inventory;
        _logger = logger;
        _tax = tax;
        _discount = discount;
        _fraud = fraud;
    }

    // This class is doing too much
}
```

> [!tip] Heuristic
> If a class takes more than 3-4 constructor dependencies (excluding cross-cutting concerns like `ILogger`), consider whether it should be split. Each dependency is a signal of an additional responsibility.

### 2. Methods That Don't Use the Same Fields

In a cohesive class, most methods operate on the same set of fields. If you can split the class's methods into two groups where each group uses a different subset of fields, you likely have two responsibilities combined.

```csharp
public class UserManager
{
    // Group 1 fields -- used by auth methods
    private readonly IPasswordHasher _hasher;
    private readonly ITokenProvider _tokens;

    // Group 2 fields -- used by profile methods
    private readonly IUserRepository _repo;
    private readonly IImageResizer _imageResizer;

    // Group 1 methods
    public bool Authenticate(string email, string password) { /* uses _hasher, _tokens */ }
    public string GenerateResetToken(string email) { /* uses _tokens */ }

    // Group 2 methods
    public void UpdateProfile(int userId, ProfileDto dto) { /* uses _repo */ }
    public void UploadAvatar(int userId, Stream image) { /* uses _repo, _imageResizer */ }
}
```

Methods `Authenticate` and `GenerateResetToken` never touch `_repo` or `_imageResizer`. Methods `UpdateProfile` and `UploadAvatar` never touch `_hasher` or `_tokens`. This class is two classes pretending to be one.

### 3. Vague Class Names

Names like `Manager`, `Handler`, `Processor`, `Helper`, `Utility`, `Service`, and `Common` are SRP red flags. They are so vague that anything can be dumped into them. A `UserManager` can manage authentication, profiles, preferences, notifications, and billing -- the name does not constrain what goes in.

Compare:
- `UserManager` -- what *doesn't* this manage?
- `UserAuthenticator` -- authenticates users. Clear boundary.
- `UserProfileRepository` -- reads/writes user profile data. Clear boundary.
- `PasswordResetService` -- handles password reset flow. Clear boundary.

> [!tip] The "And" Test
> Describe what the class does in one sentence. If you need the word "and," you likely have multiple responsibilities.
>
> - "This class validates orders **and** sends confirmation emails **and** updates inventory" -- 3 responsibilities.
> - "This class calculates shipping costs based on weight and distance" -- 1 responsibility (the "and" connects inputs, not separate concerns).

### 4. The Class Changes for Unrelated Reasons

Look at the git log for a class. If it changes because of UI redesigns *and* because of database schema changes *and* because of business rule updates, it serves multiple actors and violates SRP.

### 5. The Class Is Disproportionately Large

In C#, SRP-compliant classes typically range from 50-200 lines. Classes exceeding 300-500 lines almost always have accumulated multiple responsibilities. This is not a hard rule -- a complex algorithm might legitimately be 400 lines -- but it is a reliable signal worth investigating.

## 🔹 Violation Example: UserService God Class

Here is a realistic violation that appears in many codebases:

```csharp
public class UserService
{
    private readonly DbContext _db;
    private readonly SmtpClient _smtp;
    private readonly IConfiguration _config;

    public UserService(DbContext db, SmtpClient smtp, IConfiguration config)
    {
        _db = db;
        _smtp = smtp;
        _config = config;
    }

    // Responsibility 1: User Registration (Actor: Product team)
    public User Register(string email, string password)
    {
        // Validate input
        if (string.IsNullOrWhiteSpace(email))
            throw new ArgumentException("Email is required.");
        if (password.Length < 8)
            throw new ArgumentException("Password must be at least 8 characters.");

        // Hash password
        byte[] salt = RandomNumberGenerator.GetBytes(16);
        byte[] hash = Rfc2898DeriveBytes.Pbkdf2(
            password,
            salt,
            iterations: 100_000,
            HashAlgorithmName.SHA256,
            outputLength: 32);

        // Create user
        var user = new User
        {
            Email = email,
            PasswordHash = Convert.ToBase64String(hash),
            PasswordSalt = Convert.ToBase64String(salt),
            CreatedAt = DateTime.UtcNow
        };

        // Save to database
        _db.Users.Add(user);
        _db.SaveChanges();

        // Send welcome email
        var message = new MailMessage(
            from: _config["Email:From"],
            to: email,
            subject: "Welcome!",
            body: $"Hello {email}, welcome to our platform!");
        _smtp.Send(message);

        // Log registration
        Console.WriteLine($"[{DateTime.UtcNow}] User registered: {email}");

        return user;
    }

    // Responsibility 2: Authentication (Actor: Security team)
    public User? Authenticate(string email, string password)
    {
        var user = _db.Users.FirstOrDefault(u => u.Email == email);
        if (user is null) return null;

        byte[] salt = Convert.FromBase64String(user.PasswordSalt);
        byte[] hash = Rfc2898DeriveBytes.Pbkdf2(
            password, salt, 100_000, HashAlgorithmName.SHA256, 32);

        if (Convert.ToBase64String(hash) != user.PasswordHash)
            return null;

        return user;
    }

    // Responsibility 3: Password Reset (Actor: Security team)
    public void SendPasswordReset(string email)
    {
        var user = _db.Users.FirstOrDefault(u => u.Email == email);
        if (user is null) return;

        var token = Guid.NewGuid().ToString();
        user.ResetToken = token;
        user.ResetTokenExpiry = DateTime.UtcNow.AddHours(1);
        _db.SaveChanges();

        var resetUrl = $"{_config["App:BaseUrl"]}/reset?token={token}";
        var message = new MailMessage(
            from: _config["Email:From"],
            to: email,
            subject: "Password Reset",
            body: $"Click here to reset your password: {resetUrl}");
        _smtp.Send(message);
    }

    // Responsibility 4: User queries (Actor: various)
    public User? GetById(int id) => _db.Users.Find(id);
    public IEnumerable<User> GetAll() => _db.Users.ToList();

    // Responsibility 5: User profile updates (Actor: Product team)
    public void UpdateProfile(int userId, string name, string bio)
    {
        var user = _db.Users.Find(userId)
            ?? throw new InvalidOperationException("User not found.");
        user.Name = name;
        user.Bio = bio;
        _db.SaveChanges();
    }
}
```

**Why this violates SRP -- four actors, four reasons to change:**

1. **Product team** -- owns registration flow and profile management. When the onboarding UX changes, this class changes.
2. **Security team** -- owns authentication and password hashing. When the hashing algorithm or iteration count needs updating, this class changes.
3. **Operations/DevOps** -- if the email provider changes from SMTP to SendGrid, this class changes.
4. **DBA/Data team** -- if the database schema or ORM changes, this class changes.

A developer updating the password hashing algorithm (security concern) must work in the same file as a developer adding a new onboarding email template (product concern). They will create merge conflicts and risk accidentally breaking each other's work.

## 🔹 Fixed Example: Separated Responsibilities

Split the God class into focused classes, each serving one actor:

### PasswordHasher (Actor: Security team)

```csharp
public interface IPasswordHasher
{
    (string Hash, string Salt) HashPassword(string password);
    bool VerifyPassword(string password, string hash, string salt);
}

public class Pbkdf2PasswordHasher : IPasswordHasher
{
    private const int Iterations = 100_000;
    private const int SaltSize = 16;
    private const int HashSize = 32;

    public (string Hash, string Salt) HashPassword(string password)
    {
        byte[] salt = RandomNumberGenerator.GetBytes(SaltSize);
        byte[] hash = Rfc2898DeriveBytes.Pbkdf2(
            password, salt, Iterations, HashAlgorithmName.SHA256, HashSize);

        return (Convert.ToBase64String(hash), Convert.ToBase64String(salt));
    }

    public bool VerifyPassword(string password, string hash, string salt)
    {
        byte[] saltBytes = Convert.FromBase64String(salt);
        byte[] computedHash = Rfc2898DeriveBytes.Pbkdf2(
            password, saltBytes, Iterations, HashAlgorithmName.SHA256, HashSize);

        return CryptographicOperations.FixedTimeEquals(
            computedHash,
            Convert.FromBase64String(hash));
    }
}
```

This class has **one reason to change**: the security team decides to change the hashing algorithm, iteration count, or salt size. Nothing else touches this class.

### UserRepository (Actor: Data/DBA team)

```csharp
public interface IUserRepository
{
    User? GetById(int id);
    User? GetByEmail(string email);
    IEnumerable<User> GetAll();
    void Add(User user);
    void Update(User user);
}

public class UserRepository : IUserRepository
{
    private readonly DbContext _db;
    public UserRepository(DbContext db) => _db = db;

    public User? GetById(int id) => _db.Users.Find(id);

    public User? GetByEmail(string email)
        => _db.Users.FirstOrDefault(u => u.Email == email);

    public IEnumerable<User> GetAll() => _db.Users.ToList();

    public void Add(User user)
    {
        _db.Users.Add(user);
        _db.SaveChanges();
    }

    public void Update(User user)
    {
        _db.Users.Update(user);
        _db.SaveChanges();
    }
}
```

This class has **one reason to change**: the database schema, ORM, or data access strategy changes. If you switch from EF Core to Dapper, only this class is affected.

### EmailService (Actor: Operations/DevOps team)

```csharp
public interface IEmailService
{
    void SendWelcomeEmail(string toEmail);
    void SendPasswordResetEmail(string toEmail, string resetUrl);
}

public class SmtpEmailService : IEmailService
{
    private readonly SmtpClient _smtp;
    private readonly string _fromAddress;

    public SmtpEmailService(SmtpClient smtp, IConfiguration config)
    {
        _smtp = smtp;
        _fromAddress = config["Email:From"]!;
    }

    public void SendWelcomeEmail(string toEmail)
    {
        var message = new MailMessage(
            from: _fromAddress,
            to: toEmail,
            subject: "Welcome!",
            body: $"Hello {toEmail}, welcome to our platform!");
        _smtp.Send(message);
    }

    public void SendPasswordResetEmail(string toEmail, string resetUrl)
    {
        var message = new MailMessage(
            from: _fromAddress,
            to: toEmail,
            subject: "Password Reset",
            body: $"Click here to reset your password: {resetUrl}");
        _smtp.Send(message);
    }
}
```

This class has **one reason to change**: the email delivery mechanism changes (e.g., switch from `SmtpClient` to SendGrid, Mailgun, or AWS SES). You create a `SendGridEmailService : IEmailService` and swap it in the DI container. Nothing else in the system changes.

### UserRegistrationService (Actor: Product team)

```csharp
public class UserRegistrationService
{
    private readonly IUserRepository _userRepo;
    private readonly IPasswordHasher _hasher;
    private readonly IEmailService _email;
    private readonly ILogger<UserRegistrationService> _logger;

    public UserRegistrationService(
        IUserRepository userRepo,
        IPasswordHasher hasher,
        IEmailService email,
        ILogger<UserRegistrationService> logger)
    {
        _userRepo = userRepo;
        _hasher = hasher;
        _email = email;
        _logger = logger;
    }

    public User Register(string email, string password)
    {
        if (string.IsNullOrWhiteSpace(email))
            throw new ArgumentException("Email is required.", nameof(email));
        if (password.Length < 8)
            throw new ArgumentException(
                "Password must be at least 8 characters.", nameof(password));

        var existing = _userRepo.GetByEmail(email);
        if (existing is not null)
            throw new InvalidOperationException("Email already registered.");

        var (hash, salt) = _hasher.HashPassword(password);

        var user = new User
        {
            Email = email,
            PasswordHash = hash,
            PasswordSalt = salt,
            CreatedAt = DateTime.UtcNow
        };

        _userRepo.Add(user);
        _email.SendWelcomeEmail(email);
        _logger.LogInformation("User registered: {Email}", email);

        return user;
    }
}
```

This class **orchestrates** the registration flow. It does not know *how* passwords are hashed, *how* users are persisted, or *how* emails are sent. It only knows the *sequence of steps*. Its single reason to change: the product team decides the registration flow should include additional steps (e.g., email verification, terms acceptance, referral tracking).

### AuthenticationService (Actor: Security team)

```csharp
public class AuthenticationService
{
    private readonly IUserRepository _userRepo;
    private readonly IPasswordHasher _hasher;
    private readonly ILogger<AuthenticationService> _logger;

    public AuthenticationService(
        IUserRepository userRepo,
        IPasswordHasher hasher,
        ILogger<AuthenticationService> logger)
    {
        _userRepo = userRepo;
        _hasher = hasher;
        _logger = logger;
    }

    public User? Authenticate(string email, string password)
    {
        var user = _userRepo.GetByEmail(email);
        if (user is null)
        {
            _logger.LogWarning("Authentication failed: unknown email {Email}", email);
            return null;
        }

        if (!_hasher.VerifyPassword(password, user.PasswordHash, user.PasswordSalt))
        {
            _logger.LogWarning("Authentication failed: invalid password for {Email}", email);
            return null;
        }

        _logger.LogInformation("User authenticated: {Email}", email);
        return user;
    }
}
```

### DI Wiring in ASP.NET Core

```csharp
// In Program.cs or Startup.cs
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IPasswordHasher, Pbkdf2PasswordHasher>();
builder.Services.AddScoped<IEmailService, SmtpEmailService>();
builder.Services.AddScoped<UserRegistrationService>();
builder.Services.AddScoped<AuthenticationService>();
```

Each class is independently replaceable. Switch from SMTP to SendGrid? Register `SendGridEmailService` instead. Switch from PBKDF2 to Argon2? Register `Argon2PasswordHasher` instead. No other class changes.

## 🔹 More Examples

### ReportGenerator Splitting

**Violation:** A single class that fetches data, formats it, and exports it:

```csharp
// SRP violation: three responsibilities in one class
public class ReportGenerator
{
    private readonly string _connectionString;

    public ReportGenerator(string connectionString)
    {
        _connectionString = connectionString;
    }

    public byte[] GenerateMonthlySalesReport(int year, int month)
    {
        // Responsibility 1: Data fetching
        using var conn = new SqlConnection(_connectionString);
        conn.Open();
        var cmd = new SqlCommand(
            "SELECT ProductName, SUM(Amount) as Total FROM Sales " +
            "WHERE YEAR(SaleDate) = @year AND MONTH(SaleDate) = @month " +
            "GROUP BY ProductName ORDER BY Total DESC", conn);
        cmd.Parameters.AddWithValue("@year", year);
        cmd.Parameters.AddWithValue("@month", month);

        var data = new List<(string Product, decimal Total)>();
        using var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            data.Add((reader.GetString(0), reader.GetDecimal(1)));
        }

        // Responsibility 2: Formatting
        var sb = new StringBuilder();
        sb.AppendLine($"Monthly Sales Report - {year}/{month:D2}");
        sb.AppendLine(new string('=', 50));
        foreach (var (product, total) in data)
        {
            sb.AppendLine($"{product,-30} {total,15:C}");
        }
        sb.AppendLine(new string('=', 50));
        sb.AppendLine($"{"Grand Total",-30} {data.Sum(d => d.Total),15:C}");

        // Responsibility 3: Export to PDF
        var document = new PdfDocument();
        var page = document.AddPage();
        var gfx = XGraphics.FromPdfPage(page);
        var font = new XFont("Courier New", 10);
        gfx.DrawString(sb.ToString(), font, XBrushes.Black, 40, 40);

        using var ms = new MemoryStream();
        document.Save(ms);
        return ms.ToArray();
    }
}
```

**Fixed:** Three focused classes:

```csharp
// Responsibility 1: Data Access
public interface ISalesDataProvider
{
    IReadOnlyList<SalesLineItem> GetMonthlySales(int year, int month);
}

public class SqlSalesDataProvider : ISalesDataProvider
{
    private readonly string _connectionString;
    public SqlSalesDataProvider(string connectionString) => _connectionString = connectionString;

    public IReadOnlyList<SalesLineItem> GetMonthlySales(int year, int month)
    {
        using var conn = new SqlConnection(_connectionString);
        conn.Open();
        var cmd = new SqlCommand(
            "SELECT ProductName, SUM(Amount) as Total FROM Sales " +
            "WHERE YEAR(SaleDate) = @year AND MONTH(SaleDate) = @month " +
            "GROUP BY ProductName ORDER BY Total DESC", conn);
        cmd.Parameters.AddWithValue("@year", year);
        cmd.Parameters.AddWithValue("@month", month);

        var results = new List<SalesLineItem>();
        using var reader = cmd.ExecuteReader();
        while (reader.Read())
        {
            results.Add(new SalesLineItem(reader.GetString(0), reader.GetDecimal(1)));
        }
        return results;
    }
}

// Responsibility 2: Formatting
public interface IReportFormatter
{
    string Format(string title, IReadOnlyList<SalesLineItem> data);
}

public class TextReportFormatter : IReportFormatter
{
    public string Format(string title, IReadOnlyList<SalesLineItem> data)
    {
        var sb = new StringBuilder();
        sb.AppendLine(title);
        sb.AppendLine(new string('=', 50));
        foreach (var item in data)
        {
            sb.AppendLine($"{item.Product,-30} {item.Total,15:C}");
        }
        sb.AppendLine(new string('=', 50));
        sb.AppendLine($"{"Grand Total",-30} {data.Sum(d => d.Total),15:C}");
        return sb.ToString();
    }
}

// Responsibility 3: Export
public interface IReportExporter
{
    byte[] Export(string formattedContent);
}

public class PdfReportExporter : IReportExporter
{
    public byte[] Export(string formattedContent)
    {
        var document = new PdfDocument();
        var page = document.AddPage();
        var gfx = XGraphics.FromPdfPage(page);
        var font = new XFont("Courier New", 10);
        gfx.DrawString(formattedContent, font, XBrushes.Black, 40, 40);

        using var ms = new MemoryStream();
        document.Save(ms);
        return ms.ToArray();
    }
}

// Orchestrator: composes the three concerns
public class MonthlySalesReportService
{
    private readonly ISalesDataProvider _dataProvider;
    private readonly IReportFormatter _formatter;
    private readonly IReportExporter _exporter;

    public MonthlySalesReportService(
        ISalesDataProvider dataProvider,
        IReportFormatter formatter,
        IReportExporter exporter)
    {
        _dataProvider = dataProvider;
        _formatter = formatter;
        _exporter = exporter;
    }

    public byte[] Generate(int year, int month)
    {
        var data = _dataProvider.GetMonthlySales(year, month);
        var formatted = _formatter.Format(
            $"Monthly Sales Report - {year}/{month:D2}", data);
        return _exporter.Export(formatted);
    }
}
```

Now you can:
- Switch from SQL Server to a REST API without touching formatting or export logic.
- Switch from PDF to Excel export without touching data fetching or formatting.
- Switch from text formatting to HTML formatting without touching data or export.
- Test each concern independently with simple mocks.

### Controller Cleanup

**Violation:** A fat ASP.NET Core controller that does everything inline:

```csharp
// SRP violation: controller handles validation, business logic,
// persistence, and response formatting
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly DbContext _db;
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(DbContext db, ILogger<ProductsController> logger)
    {
        _db = db;
        _logger = logger;
    }

    [HttpPost]
    public IActionResult Create([FromBody] CreateProductRequest request)
    {
        // Validation logic (should be in a validator)
        if (string.IsNullOrWhiteSpace(request.Name))
            return BadRequest("Name is required.");
        if (request.Price <= 0)
            return BadRequest("Price must be positive.");
        if (request.Name.Length > 200)
            return BadRequest("Name must be 200 characters or less.");

        // Business logic (should be in a service)
        var sku = $"SKU-{DateTime.UtcNow:yyyyMMdd}-{Guid.NewGuid():N}"[..20];

        // Persistence (should be in a repository)
        var product = new Product
        {
            Name = request.Name,
            Price = request.Price,
            Sku = sku,
            CreatedAt = DateTime.UtcNow
        };
        _db.Products.Add(product);
        _db.SaveChanges();

        _logger.LogInformation("Product created: {Sku} - {Name}", sku, request.Name);

        // Response mapping (should be in a mapper)
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, new
        {
            product.Id, product.Name, product.Price, product.Sku, product.CreatedAt
        });
    }

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _db.Products.Find(id);
        if (product is null) return NotFound();
        return Ok(new { product.Id, product.Name, product.Price, product.Sku });
    }
}
```

**Fixed:** The controller delegates to focused classes:

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpPost]
    public IActionResult Create([FromBody] CreateProductRequest request)
    {
        var product = _productService.Create(request);
        return CreatedAtAction(nameof(GetById), new { id = product.Id }, product);
    }

    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        var product = _productService.GetById(id);
        if (product is null) return NotFound();
        return Ok(product);
    }
}
```

The controller's **single responsibility** is now HTTP concern management: routing, model binding, status codes, and response formatting. All business logic, validation, and persistence live in the service layer.

## 🔹 SRP at Different Levels

SRP applies recursively at every level of abstraction. The principle scales from individual methods all the way up to entire microservices.

### Method Level

A method should do one thing at one level of abstraction. If a method validates input, calculates a result, saves to the database, and sends an email, it is doing four things. Extract each into a separate method.

```csharp
// Violation: method does multiple things at different abstraction levels
public void ProcessOrder(Order order)
{
    // Low-level validation
    if (order.Items.Count == 0)
        throw new InvalidOperationException("No items.");
    if (order.Items.Any(i => i.Quantity <= 0))
        throw new InvalidOperationException("Bad quantity.");

    // Business calculation
    decimal subtotal = order.Items.Sum(i => i.Price * i.Quantity);
    decimal tax = subtotal * 0.08m;
    order.Total = subtotal + tax;

    // Persistence
    _db.Orders.Add(order);
    _db.SaveChanges();

    // Notification
    _email.Send(order.CustomerEmail, "Order Confirmed", $"Your total is {order.Total:C}");
}

// Fixed: each method does one thing at one level of abstraction
public void ProcessOrder(Order order)
{
    Validate(order);
    CalculateTotal(order);
    Save(order);
    NotifyCustomer(order);
}

private void Validate(Order order) { /* ... */ }
private void CalculateTotal(Order order) { /* ... */ }
private void Save(Order order) { /* ... */ }
private void NotifyCustomer(Order order) { /* ... */ }
```

### Class Level

This is the primary level where SRP is discussed. Each class serves one actor and has one reason to change. This is what most of this note covers.

### Module/Namespace Level

In C#, namespaces and project structures should group classes by responsibility:

```
MyApp.Domain/           -- business entities and rules
MyApp.Application/      -- use cases and orchestration
MyApp.Infrastructure/   -- data access, external services, email
MyApp.API/              -- controllers, middleware, HTTP concerns
```

Each project/namespace has one high-level concern. If `MyApp.Infrastructure` starts containing business rules, SRP is violated at the module level.

### Microservice Level

At the largest scale, SRP applies to entire services. A microservice should own one business capability. An `OrderService` that also manages inventory, customer profiles, and payment processing is a distributed monolith -- it has the coupling problems of a monolith without the deployment benefits.

| Level | SRP Unit | Example Violation |
|-------|----------|-------------------|
| Method | A method | `ProcessOrder` that validates, calculates, saves, and emails |
| Class | A class | `UserService` that handles auth, profiles, emails, and persistence |
| Module | A namespace/project | `Infrastructure` project containing business rules |
| Service | A microservice | `OrderService` that also manages inventory and payments |

## 🔹 The "Axis of Change" Concept

The "axis of change" is another way to think about SRP. Every class sits along one or more axes -- each axis represents a dimension along which the class might need to change. SRP says a class should sit along **exactly one axis**.

Common axes of change in enterprise C# applications:

| Axis | What Changes | Example |
|------|-------------|---------|
| **Business rules** | The domain logic itself | Tax calculation formulas, discount rules, eligibility criteria |
| **Persistence** | How and where data is stored | Database schema, ORM, switching from SQL to NoSQL |
| **Presentation** | How information is displayed | UI format, API response shape, report layout |
| **Integration** | External service contracts | Third-party API versions, message formats, auth tokens |
| **Cross-cutting** | Infrastructure concerns | Logging format, caching strategy, retry policies |

A class that calculates tax *and* saves to the database sits on two axes (business rules + persistence). When the tax formula changes, you edit the class. When the database schema changes, you edit the same class. These are independent axes -- they should be in separate classes.

```csharp
// Sits on two axes: business rules + persistence
public class TaxService
{
    private readonly DbContext _db;

    public decimal CalculateTax(Order order)
    {
        decimal rate = _db.TaxRates.First(r => r.State == order.State).Rate;
        return order.Subtotal * rate;
    }
}

// Fixed: each class sits on one axis
public class TaxCalculator  // Axis: business rules
{
    private readonly ITaxRateProvider _rates;
    public TaxCalculator(ITaxRateProvider rates) => _rates = rates;

    public decimal CalculateTax(string state, decimal subtotal)
    {
        decimal rate = _rates.GetRate(state);
        return subtotal * rate;
    }
}

public class SqlTaxRateProvider : ITaxRateProvider  // Axis: persistence
{
    private readonly DbContext _db;
    public SqlTaxRateProvider(DbContext db) => _db = db;

    public decimal GetRate(string state)
        => _db.TaxRates.First(r => r.State == state).Rate;
}
```

## 🔹 Common Mistakes

### Nano-Classes (Over-Splitting)

Taking SRP too far produces an explosion of tiny classes with a single method each. Every class needs an interface, every interface needs a DI registration, and tracing the execution flow requires jumping through 15 files.

```csharp
// Over-split: every trivial step is its own class
public class EmailAddressValidator { /* validates email format */ }
public class PasswordLengthValidator { /* checks length >= 8 */ }
public class PasswordComplexityValidator { /* checks special chars */ }
public class PasswordHistoryValidator { /* checks against history */ }
public class UserExistenceChecker { /* checks if user exists */ }
public class UserFactory { /* creates User object */ }
public class WelcomeEmailComposer { /* builds email body */ }
public class RegistrationLogger { /* logs registration event */ }
public class RegistrationOrchestrator { /* calls all the above in order */ }
```

This is SRP taken to an absurd extreme. A developer trying to understand what happens when a user registers must navigate 9 files. The cognitive overhead exceeds the benefit.

**The fix:** Group closely related operations that serve the same actor and change for the same reason. All password validation rules serve the security team and change together -- they belong in one `PasswordValidator` class. The email composition and sending both serve the DevOps/communications team -- they can live in one `EmailService` class.

### Confusing "One Thing" with "One Responsibility"

A class can do multiple *actions* while having a single *responsibility*. A `JsonSerializer` reads tokens, builds object graphs, handles escape sequences, manages recursion depth, and writes output buffers. That is many actions, but one responsibility: serialization. It serves one actor, and it changes for one reason (JSON format or serialization behavior requirements change).

The question is never "how many things does this class do?" but "how many independent reasons could this class need to change, and who would request those changes?"

### Splitting by Technical Layer Instead of Business Concern

A common mistake is creating a `Validators` class that validates everything, a `Repositories` class that persists everything, and a `Services` class that orchestrates everything. This splits by technical layer but still violates SRP within each layer -- `Validators` changes when order validation changes *and* when user validation changes.

```csharp
// Wrong: split by layer, not by concern
public class Validators
{
    public bool ValidateOrder(Order order) { /* ... */ }
    public bool ValidateUser(User user) { /* ... */ }
    public bool ValidateProduct(Product product) { /* ... */ }
}

// Right: split by concern
public class OrderValidator { /* ... */ }
public class UserValidator { /* ... */ }
public class ProductValidator { /* ... */ }
```

## 🔹 Practical Tips

**1. Start with the "who" question.** Before writing a class, ask: "Who would request a change to this class?" If you can identify more than one actor, consider splitting.

**2. Name classes after their responsibility.** If you struggle to name a class without using "Manager," "Handler," or "Service," it might be doing too much. Specific names force specific responsibilities: `PasswordHasher`, `InvoiceRenderer`, `ShippingCostCalculator`.

**3. Watch the constructor.** More than 3-4 dependencies (excluding `ILogger`) is a strong signal. Each dependency is a relationship to another concern.

**4. Use the "newspaper test."** Robert C. Martin suggests that a well-organized class reads like a newspaper article: the headline (class name) tells you what it is about, the leading paragraph (public methods) gives the overview, and the details (private methods) follow. If the "headline" is too vague to be useful, the class is too broad.

**5. Refactor incrementally.** You do not need to achieve perfect SRP on the first pass. Start with a working class, identify the axes of change as they emerge over time, and extract classes when the need becomes clear. The [[2 - Open-Closed Principle|Open/Closed Principle]] and [[5 - Dependency Inversion Principle|Dependency Inversion Principle]] make this refactoring safe.

**6. SRP is relative to change frequency.** In a codebase where the database never changes and the email provider never changes, it may be fine to have persistence and email logic in the same class. SRP is most valuable along the axes where change is **frequent** and **independent**. Focus your splitting effort where the pain is.

**7. Cross-cutting concerns are exempt.** Logging, caching, and error handling appear in many classes. This does not violate SRP -- these are cross-cutting concerns, not core responsibilities. Using `ILogger` in a `PaymentProcessor` does not make `PaymentProcessor` responsible for logging. The responsibility of `PaymentProcessor` is payment processing; it *uses* logging as a tool.
