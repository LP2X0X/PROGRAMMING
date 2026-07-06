---
tags: [csharp, asp-net-core, environments, configuration]
---


You can register different service implementations based on the environment. This is useful for swapping real services with fakes or mocks during development.

### Example: Email Service

```csharp
var builder = WebApplication.CreateBuilder(args);

if (builder.Environment.IsDevelopment())
{
    // In development, log emails to the console instead of sending them
    builder.Services.AddSingleton<IEmailSender, ConsoleEmailSender>();
}
else
{
    // In staging and production, use the real SMTP sender
    builder.Services.AddSingleton<IEmailSender, SmtpEmailSender>();
}
```

```csharp
public interface IEmailSender
{
    Task SendAsync(string to, string subject, string body);
}

public class ConsoleEmailSender : IEmailSender
{
    public Task SendAsync(string to, string subject, string body)
    {
        Console.WriteLine("========== EMAIL ==========");
        Console.WriteLine($"To: {to}");
        Console.WriteLine($"Subject: {subject}");
        Console.WriteLine($"Body: {body}");
        Console.WriteLine("===========================");
        return Task.CompletedTask;
    }
}

public class SmtpEmailSender : IEmailSender
{
    private readonly EmailSettings _settings;

    public SmtpEmailSender(IOptions<EmailSettings> settings)
    {
        _settings = settings.Value;
    }

    public async Task SendAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_settings.SmtpServer, _settings.Port);
        client.EnableSsl = _settings.EnableSsl;
        var message = new MailMessage("noreply@company.com", to, subject, body);
        await client.SendMailAsync(message);
    }
}
```

### Example: Payment Gateway

```csharp
if (builder.Environment.IsDevelopment() || builder.Environment.IsStaging())
{
    // Use a sandbox/mock payment processor
    builder.Services.AddSingleton<IPaymentGateway, SandboxPaymentGateway>();
}
else
{
    // Use the live payment processor
    builder.Services.AddSingleton<IPaymentGateway, StripePaymentGateway>();
}
```

### Example: Logging Configuration

```csharp
if (builder.Environment.IsDevelopment())
{
    builder.Services.AddLogging(logging =>
    {
        logging.AddConsole();
        logging.AddDebug();
        logging.SetMinimumLevel(LogLevel.Debug);
    });
}
else
{
    builder.Services.AddLogging(logging =>
    {
        logging.AddConsole();
        logging.SetMinimumLevel(LogLevel.Warning);
        // Add a structured logging sink for production
        // logging.AddApplicationInsights();
    });
}
```

> [!summary] Section Summary
> - Use `builder.Environment.IsDevelopment()` to register different service implementations per environment.
> - Common pattern: mock/console services in Development, real services in Production.
> - Payment gateways, email senders, and logging are typical candidates for environment-based swapping.
