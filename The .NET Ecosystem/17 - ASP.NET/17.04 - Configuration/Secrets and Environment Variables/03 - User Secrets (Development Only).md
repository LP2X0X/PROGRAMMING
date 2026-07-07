---
tags:
  - csharp
  - asp-net-core
  - configuration
  - secrets
  - security
---


**User Secrets** is a built-in ASP.NET Core feature that stores sensitive configuration in a JSON file outside the project directory -- in the developer's user profile folder. This means the secrets never appear in the project tree and cannot be accidentally committed.

### How It Works

User Secrets associates a project with a unique GUID (the **UserSecretsId**). At runtime, the configuration system reads from a JSON file keyed to that GUID, stored in the user's profile directory. The values from this file override values from `appsettings.json` and `appsettings.Development.json`.

> [!info] Definition
> **User Secrets** is a development-time configuration provider that stores key-value pairs in a JSON file located in the operating system's user profile directory, completely outside the project folder.

### Initializing User Secrets

Run the following command from the project directory (where the `.csproj` file lives):

```bash
dotnet user-secrets init
```

This adds a `UserSecretsId` element to your `.csproj` file:

```xml
<PropertyGroup>
  <TargetFramework>net8.0</TargetFramework>
  <UserSecretsId>a1b2c3d4-e5f6-7890-abcd-ef1234567890</UserSecretsId>
</PropertyGroup>
```

> [!tip] Pro Tip
> The GUID is just an identifier -- it has no cryptographic significance. Its sole purpose is to link your project to a specific secrets file on disk. You can share the same GUID across multiple projects if you want them to share secrets (though this is rarely useful).

### Setting Secrets

```bash
# Set a single secret (hierarchical keys use colon separator)
dotnet user-secrets set "Smtp:Password" "my-secret-password"

# Set a connection string
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost;Database=MyApp;User=dev;Password=dev123"

# Set an API key
dotnet user-secrets set "Stripe:SecretKey" "sk_test_abc123"
```

### Listing Secrets

```bash
dotnet user-secrets list
```

Output:

```
Stripe:SecretKey = sk_test_abc123
Smtp:Password = my-secret-password
ConnectionStrings:DefaultConnection = Server=localhost;Database=MyApp;User=dev;Password=dev123
```

### Removing Secrets

```bash
# Remove a single secret
dotnet user-secrets remove "Smtp:Password"

# Remove all secrets for this project
dotnet user-secrets clear
```

### Where the File Lives

On Windows, the secrets file is stored at:

```
%APPDATA%\Microsoft\UserSecrets\<UserSecretsId>\secrets.json
```

For example:

```
C:\Users\YourName\AppData\Roaming\Microsoft\UserSecrets\a1b2c3d4-e5f6-7890-abcd-ef1234567890\secrets.json
```

On Linux and macOS:

```
~/.microsoft/usersecrets/<UserSecretsId>/secrets.json
```

The file itself is plain JSON with the same structure as `appsettings.json`:

```json
{
  "Smtp": {
    "Password": "my-secret-password"
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyApp;User=dev;Password=dev123"
  },
  "Stripe": {
    "SecretKey": "sk_test_abc123"
  }
}
```

> [!warning] Common Misconception
> User Secrets are **not encrypted**. They are stored as plain-text JSON. The protection comes from the file living outside the project directory (so it cannot be committed to git), not from encryption. Do not rely on User Secrets for production environments.

### Already Wired Up by Default

When you create a project with `WebApplication.CreateBuilder(args)`, User Secrets are automatically added to the configuration pipeline in the **Development** environment. You do not need any extra setup code:

```csharp
var builder = WebApplication.CreateBuilder(args);

// In Development, CreateBuilder() automatically calls:
// builder.Configuration.AddUserSecrets<Program>();
// You do NOT need to add this line yourself.

// Access secrets exactly like any other configuration
var smtpPassword = builder.Configuration["Smtp:Password"];
var connectionString = builder.Configuration.GetConnectionString("DefaultConnection");
```

> [!tip] Pro Tip
> If you are using a non-web project (like a console app or worker service) and want User Secrets, you need to add the `Microsoft.Extensions.Configuration.UserSecrets` NuGet package and call `AddUserSecrets()` explicitly:
> ```csharp
> var config = new ConfigurationBuilder()
>     .AddJsonFile("appsettings.json")
>     .AddUserSecrets<Program>()
>     .Build();
> ```

### Using Secrets with the Options Pattern

User Secrets integrate seamlessly with strongly typed configuration via the [[Options Pattern]]:

```csharp
// SmtpOptions.cs
public class SmtpOptions
{
    public const string SectionName = "Smtp";

    public string Host { get; set; } = string.Empty;
    public int Port { get; set; }
    public string Username { get; set; } = string.Empty;
    public string Password { get; set; } = string.Empty;  // Comes from User Secrets
}
```

```csharp
// Program.cs
builder.Services.Configure<SmtpOptions>(
    builder.Configuration.GetSection(SmtpOptions.SectionName));
```

```json
// appsettings.json -- non-sensitive values only
{
  "Smtp": {
    "Host": "smtp.example.com",
    "Port": 587,
    "Username": "noreply@example.com"
  }
}
```

```bash
# Secret stored via User Secrets
dotnet user-secrets set "Smtp:Password" "actual-password"
```

At runtime, the configuration system merges all sources. The `SmtpOptions` object receives `Host`, `Port`, and `Username` from `appsettings.json` and `Password` from User Secrets.

> [!summary] Section Summary
> User Secrets store sensitive configuration in a plain-text JSON file in the user profile directory, outside the project tree. Initialize with `dotnet user-secrets init`, manage with `set`, `list`, `remove`, and `clear`. Already wired up by default in `CreateBuilder()` for Development. Not encrypted -- protection comes from file location, not encryption.
