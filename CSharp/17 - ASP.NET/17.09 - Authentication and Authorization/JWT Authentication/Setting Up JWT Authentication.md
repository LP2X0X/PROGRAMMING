---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


## Required NuGet Package

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

## Service Registration in Program.cs

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();
```

## Explanation of Each Validation Parameter

| Parameter | Purpose |
|---|---|
| `ValidateIssuer` | Checks that the `iss` claim matches `ValidIssuer`. Prevents tokens from untrusted issuers. |
| `ValidateAudience` | Checks that the `aud` claim matches `ValidAudience`. Ensures the token was intended for your API. |
| `ValidateLifetime` | Rejects expired tokens by checking the `exp` claim against the current time. |
| `ValidateIssuerSigningKey` | Verifies the token's signature using the provided key. This is the most critical check -- it proves the token was not tampered with. |
| `ValidIssuer` | The expected value of the `iss` claim (e.g., `"https://myapi.com"`). |
| `ValidAudience` | The expected value of the `aud` claim (e.g., `"https://myapi.com"`). |
| `IssuerSigningKey` | The cryptographic key used to validate the token's signature. |

> [!danger]
> **Never** set `ValidateIssuerSigningKey = false` in production. Without signature validation, anyone can forge tokens and impersonate any user. The other validation parameters add defense-in-depth, but the signing key validation is the core security mechanism.

> [!tip]
> The middleware order matters. `UseAuthentication()` must come **before** `UseAuthorization()`. If reversed, authorization checks run before the user identity is established and every request will be treated as unauthenticated.

> [!summary] Section Summary
> JWT authentication in ASP.NET Core is configured by adding the `JwtBearer` authentication scheme and specifying `TokenValidationParameters`. Each parameter controls a specific aspect of token verification.
