---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


ASP.NET Core encrypts authentication cookies using the **Data Protection** system. Understanding this is essential for production deployments, especially in web farm or containerized environments.

### How It Works

When you call `SignInAsync()`:

1. The `ClaimsPrincipal` is serialized into a byte array.
2. The Data Protection system encrypts and signs the byte array using a key from the key ring.
3. The encrypted bytes are Base64-encoded and set as the cookie value.

When the cookie comes back on a request:

1. The middleware reads the cookie value.
2. The Data Protection system decrypts and verifies the signature.
3. The bytes are deserialized back into a `ClaimsPrincipal`.

### Key Storage

By default, Data Protection keys are stored at:

- **Windows**: `%LOCALAPPDATA%\ASP.NET\DataProtection-Keys`
- **Linux/macOS**: `~/.aspnet/DataProtection-Keys`

> [!danger] Security Warning -- Web Farms
> In a web farm (multiple servers behind a load balancer), each server has its own key ring by default. A cookie encrypted by Server A cannot be decrypted by Server B. This causes users to be randomly logged out when their requests hit different servers. You must configure **shared key storage** for web farms.

### Configuring Shared Key Storage

```csharp
// Store keys in a shared location for web farm scenarios
builder.Services.AddDataProtection()
    .PersistKeysToFileSystem(new DirectoryInfo(@"\\server\share\keys"))
    .SetApplicationName("MyApp");

// Or use Azure Blob Storage
builder.Services.AddDataProtection()
    .PersistKeysToAzureBlobStorage(connectionString, containerName, blobName)
    .ProtectKeysWithAzureKeyVault(keyUri, tokenCredential);

// Or use Redis
builder.Services.AddDataProtection()
    .PersistKeysToStackExchangeRedis(connectionMultiplexer, "DataProtection-Keys");
```

> [!tip] SetApplicationName
> When multiple applications share the same key storage, call `SetApplicationName("MyApp")` to isolate each app's keys. Without this, one app could accidentally decrypt another app's cookies.

> [!summary] Section Summary
> Cookies are encrypted using ASP.NET Core Data Protection. Keys are stored locally by default. For web farms, configure shared key storage (file system, database, Redis, or cloud storage) so all servers can decrypt the same cookies.
