---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


Here is a detailed reference of all commonly used `TokenValidationParameters`:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `ValidIssuer` | `string` | `null` | The expected issuer. Compared against the `iss` claim. |
| `ValidIssuers` | `IEnumerable<string>` | `null` | Multiple valid issuers (useful when migrating between identity providers). |
| `ValidAudience` | `string` | `null` | The expected audience. Compared against the `aud` claim. |
| `ValidAudiences` | `IEnumerable<string>` | `null` | Multiple valid audiences (useful for APIs serving multiple clients). |
| `IssuerSigningKey` | `SecurityKey` | `null` | The key used to validate the signature. |
| `IssuerSigningKeys` | `IEnumerable<SecurityKey>` | `null` | Multiple valid keys (essential for key rotation). |
| `ValidateLifetime` | `bool` | `true` | Whether to reject expired tokens. |
| `ClockSkew` | `TimeSpan` | `5 minutes` | Tolerance for clock differences between servers. |
| `RequireExpirationTime` | `bool` | `true` | Rejects tokens without an `exp` claim. |
| `RequireSignedTokens` | `bool` | `true` | Rejects unsigned tokens. |

> [!warning] Common Misconception
> The default `ClockSkew` of 5 minutes means a token remains valid for up to 5 minutes **after** its `exp` time. In high-security environments, you may want to reduce this to `TimeSpan.Zero`, but only if your server clocks are well-synchronized (e.g., using NTP).

```csharp
options.TokenValidationParameters = new TokenValidationParameters
{
    // ... other parameters ...
    ClockSkew = TimeSpan.Zero // No tolerance for expired tokens
};
```

> [!summary] Section Summary
> `TokenValidationParameters` offers fine-grained control over every aspect of token validation. Pay special attention to `ClockSkew` (defaults to 5 minutes) and `RequireExpirationTime` (defaults to true).
