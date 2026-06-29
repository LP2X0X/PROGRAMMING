---
tags:
  - csharp
  - asp-net-core
  - configuration
  - options-pattern
---


### Example 1: JWT Authentication Settings

```json
{
  "Jwt": {
    "Issuer": "https://myapp.example.com",
    "Audience": "https://api.myapp.example.com",
    "SecretKey": "your-256-bit-secret-key-here-min-32-chars!",
    "ExpirationMinutes": 60,
    "RefreshExpirationDays": 7,
    "AllowedAlgorithms": [ "HS256", "HS384" ]
  }
}
```

```csharp
public class JwtSettings
{
    public const string SectionName = "Jwt";

    [Required]
    [Url]
    public string Issuer { get; set; } = string.Empty;

    [Required]
    [Url]
    public string Audience { get; set; } = string.Empty;

    [Required]
    [MinLength(32, ErrorMessage = "Secret key must be at least 32 characters for HS256")]
    public string SecretKey { get; set; } = string.Empty;

    [Range(1, 1440)]
    public int ExpirationMinutes { get; set; } = 60;

    [Range(1, 90)]
    public int RefreshExpirationDays { get; set; } = 7;

    public List<string> AllowedAlgorithms { get; set; } = new() { "HS256" };
}
```

```csharp
// Program.cs
builder.Services.AddOptions<JwtSettings>()
    .Bind(builder.Configuration.GetSection(JwtSettings.SectionName))
    .ValidateDataAnnotations()
    .Validate(jwt =>
    {
        if (jwt.RefreshExpirationDays * 24 * 60 <= jwt.ExpirationMinutes)
            return false;
        return true;
    }, "Refresh token expiration must be longer than access token expiration")
    .ValidateOnStart();
```

```csharp
// TokenService.cs
public class TokenService
{
    private readonly JwtSettings _jwt;

    public TokenService(IOptions<JwtSettings> options)
    {
        _jwt = options.Value;
    }

    public string GenerateToken(User user)
    {
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_jwt.SecretKey));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new Claim(JwtRegisteredClaimNames.Email, user.Email),
            new Claim(ClaimTypes.Role, user.Role)
        };

        var token = new JwtSecurityToken(
            issuer: _jwt.Issuer,
            audience: _jwt.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwt.ExpirationMinutes),
            signingCredentials: credentials);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

### Example 2: Feature Flags

```json
{
  "Features": {
    "EnableDarkMode": true,
    "EnableBetaApi": false,
    "MaxUploadSizeMb": 25,
    "MaintenanceWindow": {
      "Enabled": false,
      "StartUtc": "2026-06-20T02:00:00Z",
      "EndUtc": "2026-06-20T04:00:00Z"
    }
  }
}
```

```csharp
public class FeatureFlags
{
    public const string SectionName = "Features";

    public bool EnableDarkMode { get; set; }
    public bool EnableBetaApi { get; set; }

    [Range(1, 500)]
    public int MaxUploadSizeMb { get; set; } = 10;

    public MaintenanceWindowOptions MaintenanceWindow { get; set; } = new();
}

public class MaintenanceWindowOptions
{
    public bool Enabled { get; set; }
    public DateTime? StartUtc { get; set; }
    public DateTime? EndUtc { get; set; }
}
```

```csharp
// Use IOptionsMonitor for feature flags -- they may change at runtime
public class FeatureService
{
    private readonly IOptionsMonitor<FeatureFlags> _features;

    public FeatureService(IOptionsMonitor<FeatureFlags> features)
    {
        _features = features;

        _features.OnChange(flags =>
        {
            Console.WriteLine($"Feature flags updated. Dark mode: {flags.EnableDarkMode}");
        });
    }

    public bool IsDarkModeEnabled() => _features.CurrentValue.EnableDarkMode;

    public bool IsInMaintenanceWindow()
    {
        var mw = _features.CurrentValue.MaintenanceWindow;
        if (!mw.Enabled || mw.StartUtc is null || mw.EndUtc is null)
            return false;

        var now = DateTime.UtcNow;
        return now >= mw.StartUtc && now <= mw.EndUtc;
    }
}
```

> [!tip] Pro Tip
> Feature flags are a prime use case for `IOptionsMonitor<T>`. You can toggle features by editing `appsettings.json` (or an external config source) without restarting the application.

### Example 3: Multiple Database Connections (Named Options)

```json
{
  "Databases": {
    "Primary": {
      "ConnectionString": "Server=db1;Database=AppDb;...",
      "CommandTimeout": 30,
      "EnableRetry": true
    },
    "ReadReplica": {
      "ConnectionString": "Server=db2;Database=AppDb;...",
      "CommandTimeout": 15,
      "EnableRetry": false
    }
  }
}
```

```csharp
public class DatabaseOptions
{
    [Required]
    public string ConnectionString { get; set; } = string.Empty;

    [Range(5, 300)]
    public int CommandTimeout { get; set; } = 30;

    public bool EnableRetry { get; set; } = true;
}

public static class DatabaseOptionNames
{
    public const string Primary = nameof(Primary);
    public const string ReadReplica = nameof(ReadReplica);
}
```

```csharp
// Program.cs
builder.Services.Configure<DatabaseOptions>(
    DatabaseOptionNames.Primary,
    builder.Configuration.GetSection("Databases:Primary"));

builder.Services.Configure<DatabaseOptions>(
    DatabaseOptionNames.ReadReplica,
    builder.Configuration.GetSection("Databases:ReadReplica"));

// Ensure all named instances are validated
builder.Services.PostConfigureAll<DatabaseOptions>(opts =>
{
    if (opts.CommandTimeout < 5)
        opts.CommandTimeout = 5;
});
```

```csharp
// Repository using named options
public class OrderRepository
{
    private readonly DatabaseOptions _primary;
    private readonly DatabaseOptions _readReplica;

    public OrderRepository(IOptionsSnapshot<DatabaseOptions> options)
    {
        _primary = options.Get(DatabaseOptionNames.Primary);
        _readReplica = options.Get(DatabaseOptionNames.ReadReplica);
    }

    public async Task<Order> GetByIdAsync(int id)
    {
        // Use read replica for queries
        using var conn = new SqlConnection(_readReplica.ConnectionString);
        conn.Open();
        // ...
    }

    public async Task CreateAsync(Order order)
    {
        // Use primary for writes
        using var conn = new SqlConnection(_primary.ConnectionString);
        conn.Open();
        // ...
    }
}
```

> [!summary] Section Summary
> Real-world Options pattern usage spans JWT settings (validated at startup with `IOptions<T>`), feature flags (monitored at runtime with `IOptionsMonitor<T>`), and multi-database configurations (named options with `IOptionsSnapshot<T>`). Choose the injection interface based on your reload requirements.
