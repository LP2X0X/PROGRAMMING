---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


Token generation is the process of creating a JWT to hand to the client after successful authentication.

## Step-by-Step Token Generation

```csharp
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using Microsoft.IdentityModel.Tokens;
using System.Text;

public class TokenService
{
    private readonly IConfiguration _configuration;

    public TokenService(IConfiguration configuration)
    {
        _configuration = configuration;
    }

    public string GenerateToken(string userId, string email, string role)
    {
        // Step 1: Create claims
        var claims = new List<Claim>
        {
            new Claim(JwtRegisteredClaimNames.Sub, userId),
            new Claim(JwtRegisteredClaimNames.Email, email),
            new Claim(ClaimTypes.Role, role),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString()),
            new Claim(JwtRegisteredClaimNames.Iat,
                DateTimeOffset.UtcNow.ToUnixTimeSeconds().ToString(),
                ClaimValueTypes.Integer64)
        };

        // Step 2: Create signing key and credentials
        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]!));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        // Step 3: Build the token
        var token = new JwtSecurityToken(
            issuer: _configuration["Jwt:Issuer"],
            audience: _configuration["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(30),
            signingCredentials: credentials);

        // Step 4: Serialize to string
        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

## Breakdown of Each Step

1. **Claims** -- key-value pairs embedded in the token. Use `JwtRegisteredClaimNames` for standard claims and `ClaimTypes` for ASP.NET-specific claims like `Role`.

2. **SymmetricSecurityKey** -- wraps the raw secret bytes into a key object. The key must be at least 256 bits (32 bytes) for `HS256`.

3. **SigningCredentials** -- pairs the key with the algorithm (`HmacSha256`).

4. **JwtSecurityToken** -- the token object with all its properties: issuer, audience, claims, expiration, and signing credentials.

5. **WriteToken** -- serializes the token into the compact `header.payload.signature` string format.

> [!tip]
> Always include a `jti` (JWT ID) claim with a unique GUID. This allows you to implement token revocation by maintaining a blacklist of revoked `jti` values.

> [!warning] Common Misconception
> The `Jwt:Key` in `appsettings.json` must be long enough for the chosen algorithm. For `HS256`, the key must be at least 32 characters (256 bits). A short key like `"mysecret"` will throw a runtime exception.

> [!summary] Section Summary
> Token generation involves creating claims, constructing a signing key, building a `JwtSecurityToken`, and serializing it with `JwtSecurityTokenHandler`. Always use a sufficiently long secret key and include a `jti` for revocation support.
