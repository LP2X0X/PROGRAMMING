---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


## Why Refresh Tokens Exist

Access tokens should be **short-lived** (15 to 60 minutes). If an access token is stolen, the damage is limited to its short lifetime. But forcing users to re-enter credentials every 30 minutes is unacceptable. Refresh tokens solve this by allowing the client to obtain a new access token without user interaction.

## The Refresh Token Flow

```
1. Client logs in --> Server returns Access Token (30 min) + Refresh Token (7 days)
2. Client makes API calls with Access Token
3. Access Token expires
4. Client sends Refresh Token to /api/auth/refresh
5. Server validates Refresh Token, issues new Access Token + new Refresh Token
6. Old Refresh Token is invalidated (rotation)
7. Repeat from step 2
```

> [!ad-note]
> Refresh tokens are **opaque** -- they are not JWTs. They are random strings stored in the database. This means the server can revoke them instantly by deleting them from the database, unlike JWTs which remain valid until they expire.

## Implementation

### Refresh Token Entity

```csharp
public class RefreshToken
{
    public int Id { get; set; }
    public string Token { get; set; } = string.Empty;
    public string UserId { get; set; } = string.Empty;
    public DateTime ExpiresAt { get; set; }
    public DateTime CreatedAt { get; set; }
    public bool IsRevoked { get; set; }
    public string? ReplacedByToken { get; set; }
}
```

### Updated Token Service

```csharp
public class TokenService
{
    private readonly IConfiguration _configuration;
    private readonly AppDbContext _context;

    public TokenService(IConfiguration configuration, AppDbContext context)
    {
        _configuration = configuration;
        _context = context;
    }

    public string GenerateAccessToken(string userId, string email, string role)
    {
        // Same as before -- generates a short-lived JWT
        var claims = new List<Claim>
        {
            new Claim(JwtRegisteredClaimNames.Sub, userId),
            new Claim(JwtRegisteredClaimNames.Email, email),
            new Claim(ClaimTypes.Role, role),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        var key = new SymmetricSecurityKey(
            Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]!));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _configuration["Jwt:Issuer"],
            audience: _configuration["Jwt:Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(30),
            signingCredentials: credentials);

        return new JwtSecurityTokenHandler().WriteToken(token);
    }

    public async Task<RefreshToken> GenerateRefreshTokenAsync(string userId)
    {
        var refreshToken = new RefreshToken
        {
            Token = Convert.ToBase64String(RandomNumberGenerator.GetBytes(64)),
            UserId = userId,
            ExpiresAt = DateTime.UtcNow.AddDays(7),
            CreatedAt = DateTime.UtcNow
        };

        _context.RefreshTokens.Add(refreshToken);
        await _context.SaveChangesAsync();

        return refreshToken;
    }
}
```

### Refresh Endpoint

```csharp
[HttpPost("refresh")]
public async Task<IActionResult> Refresh([FromBody] RefreshRequest request)
{
    var storedToken = await _context.RefreshTokens
        .FirstOrDefaultAsync(rt => rt.Token == request.RefreshToken);

    if (storedToken is null || storedToken.IsRevoked || storedToken.ExpiresAt < DateTime.UtcNow)
    {
        return Unauthorized(new { message = "Invalid or expired refresh token" });
    }

    // Revoke the old refresh token (rotation)
    storedToken.IsRevoked = true;

    var user = await _userService.GetByIdAsync(storedToken.UserId);
    var newAccessToken = _tokenService.GenerateAccessToken(
        user.Id.ToString(), user.Email, user.Role);
    var newRefreshToken = await _tokenService.GenerateRefreshTokenAsync(user.Id.ToString());

    storedToken.ReplacedByToken = newRefreshToken.Token;
    await _context.SaveChangesAsync();

    return Ok(new
    {
        AccessToken = newAccessToken,
        RefreshToken = newRefreshToken.Token
    });
}

public record RefreshRequest(string RefreshToken);
```

> [!tip]
> **Refresh token rotation** -- always issue a new refresh token when the old one is used, and invalidate the old one. If an attacker steals a refresh token and uses it, the legitimate user's next refresh attempt will fail (the token was already rotated), alerting the system to a potential breach.

> [!summary] Section Summary
> Refresh tokens allow users to stay authenticated without re-entering credentials. Access tokens are short-lived JWTs; refresh tokens are opaque, database-stored strings with longer lifetimes. Always implement rotation -- invalidate old refresh tokens when issuing new ones.
