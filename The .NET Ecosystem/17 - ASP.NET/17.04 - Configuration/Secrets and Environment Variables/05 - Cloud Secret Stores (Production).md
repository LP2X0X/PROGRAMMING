---
tags:
  - csharp
  - asp-net-core
  - configuration
  - secrets
  - security
---


For production environments, environment variables work but have limitations: they are visible in process listings, may appear in crash dumps, and managing hundreds of them across multiple services becomes unwieldy. **Cloud secret stores** provide encrypted storage, access control, versioning, and auditing.

### Azure Key Vault

**Azure Key Vault** is Microsoft's managed secret store. ASP.NET Core has first-class integration via the `Azure.Extensions.AspNetCore.Configuration.Secrets` NuGet package.

```bash
dotnet add package Azure.Extensions.AspNetCore.Configuration.Secrets
dotnet add package Azure.Identity
```

```csharp
var builder = WebApplication.CreateBuilder(args);

if (!builder.Environment.IsDevelopment())
{
    var keyVaultUri = new Uri(builder.Configuration["KeyVault:Uri"]!);
    builder.Configuration.AddAzureKeyVault(keyVaultUri, new DefaultAzureCredential());
}
```

Secrets stored in Key Vault with the name `Smtp--Password` (using `--` as separator) map to the configuration key `Smtp:Password`.

> [!tip] Pro Tip
> Use **Managed Identity** (`DefaultAzureCredential`) so your application authenticates to Key Vault without storing any credentials at all. The Azure platform handles identity transparently.

### AWS Secrets Manager

AWS provides **Secrets Manager** for the same purpose. Use the `AWSSDK.SecretsManager` NuGet package or community configuration providers:

```csharp
builder.Configuration.AddSecretsManager(configurator: options =>
{
    options.SecretFilter = entry => entry.Name.StartsWith("MyApp/");
    options.KeyGenerator = (_, name) => name.Replace("MyApp/", "").Replace("/", ":");
});
```

### HashiCorp Vault

For multi-cloud or on-premises scenarios, **HashiCorp Vault** is a popular choice. Community NuGet packages provide configuration provider integration.

| Feature | Azure Key Vault | AWS Secrets Manager | HashiCorp Vault |
|---|---|---|---|
| Managed service | Yes | Yes | Self-hosted or cloud |
| .NET integration | First-party NuGet | AWS SDK | Community packages |
| Secret versioning | Yes | Yes | Yes |
| Access auditing | Yes | Yes | Yes |
| Encryption | HSM-backed | AWS KMS | Configurable |

> [!summary] Section Summary
> Cloud secret stores (Azure Key Vault, AWS Secrets Manager, HashiCorp Vault) provide encrypted, audited, access-controlled secret management for production. They integrate into the ASP.NET Core configuration pipeline as additional providers.
