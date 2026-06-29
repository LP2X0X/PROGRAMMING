---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


Once registered, inject the options into your services using one of three interfaces.

### Basic Injection with IOptions

```csharp
public class EmailService
{
    private readonly SmtpSettings _settings;

    public EmailService(IOptions<SmtpSettings> options)
    {
        _settings = options.Value;
    }

    public async Task SendAsync(string to, string subject, string body)
    {
        using var client = new SmtpClient(_settings.Host, _settings.Port);
        client.EnableSsl = _settings.EnableSsl;
        client.Credentials = new NetworkCredential(
            _settings.Username, _settings.Password);

        // Full IntelliSense, type safety, no magic strings
        await client.SendMailAsync(
            new MailMessage(_settings.Username, to, subject, body));
    }
}
```

### Unit Testing with Options

One of the biggest wins: you can unit test without any configuration infrastructure.

```csharp
[Fact]
public async Task SendAsync_UsesConfiguredHost()
{
    // Arrange -- no appsettings.json, no IConfiguration, just a POCO
    var settings = new SmtpSettings
    {
        Host = "test-smtp.local",
        Port = 25,
        Username = "test@test.com",
        EnableSsl = false
    };

    var service = new EmailService(Options.Create(settings));

    // Act & Assert ...
}
```

> [!tip] Pro Tip
> `Options.Create<T>(value)` wraps any POCO in an `IOptions<T>` for testing. It lives in `Microsoft.Extensions.Options` and requires no mocking framework.

> [!summary] Section Summary
> Inject `IOptions<T>` into constructors and access `.Value` to get the POCO. For unit testing, use `Options.Create(new MySettings { ... })` to create test instances without configuration infrastructure.
