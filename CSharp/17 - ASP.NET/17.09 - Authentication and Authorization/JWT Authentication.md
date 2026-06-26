---
tags: [csharp, asp-net-core, authentication, jwt, api, security]
aliases: [JWT Auth, JSON Web Token, Bearer Token Authentication]
status: complete
date: 2026-06-18
---

# JWT Authentication

## Table of Contents

- [[#What is JWT]]
- [[#JWT Structure]]
- [[#When to Use JWT]]
- [[#Setting Up JWT Authentication]]
- [[#Token Validation Parameters]]
- [[#Generating Tokens]]
- [[#A Login Endpoint]]
- [[#Refresh Tokens]]
- [[#Storing Tokens on the Client]]
- [[#Common JWT Claims]]
- [[#Accessing Claims in Controllers]]
- [[#JWT vs Cookies Comparison]]
- [[#Security Considerations]]
- [[#Complete Real-World Example]]
- [[#Related Topics]]
- [[#Further Reading]]
- [[#Comprehensive Summary]]

---

## What is JWT

> [!info] Definition
> **JSON Web Token (JWT)** is an open standard (RFC 7519) that defines a compact, URL-safe, self-contained way of transmitting information between parties as a JSON object. The information can be verified and trusted because it is digitally signed.

JWT is the dominant authentication mechanism for modern APIs. The key concept is **self-contained** -- the token itself carries all the information needed to identify and authorize the user. Unlike traditional session-based authentication where the server stores session data in memory or a database and the client carries only a session ID, a JWT embeds the user's identity and claims directly into the token.

This means:

- The server does **not** need to look up a session store on every request
- Any server in a load-balanced cluster can validate the token independently
- The token is portable across services and domains

> [!ad-note]
> "Self-contained" means the token is both the **ticket** and the **passenger manifest**. A session cookie is just a ticket number -- you have to go to the counter (the database) to find out who it belongs to. A JWT carries the passenger's name, seat assignment, and boarding group right on the ticket itself.

A JWT is typically sent in the `Authorization` header of an HTTP request:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

> [!summary] Section Summary
> JWT is a self-contained, stateless token format for API authentication. The token carries user identity and claims, eliminating the need for server-side session storage.

---

## JWT Structure

A JWT consists of three parts separated by dots (`.`):

```
Header.Payload.Signature
```

Each part is **Base64Url** encoded. Here is a visual breakdown:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwicm9sZSI6IkFkbWluIiwiZXhwIjoxNzE4NzQwMDAwLCJpYXQiOjE3MTg3MzY0MDB9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
|___________________________|  |______________________________________________|  |___________________________________|
         Header                               Payload                                      Signature
```

### Header

The header describes the token type and the signing algorithm:

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- `alg` -- the signing algorithm. `HS256` (HMAC-SHA256) uses a shared secret. `RS256` (RSA-SHA256) uses a public/private key pair.
- `typ` -- always `"JWT"` for JSON Web Tokens.

### Payload

The payload contains **claims** -- statements about the user and additional metadata:

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "role": "Admin",
  "email": "john@example.com",
  "exp": 1718740000,
  "iat": 1718736400
}
```

- `sub` -- subject, typically the user ID
- `name` -- a custom claim with the user's name
- `role` -- a custom claim for authorization
- `exp` -- expiration time as a Unix timestamp
- `iat` -- issued-at time as a Unix timestamp

> [!warning] Common Misconception
> The payload is **encoded**, not **encrypted**. Anyone can decode the Base64Url string and read the claims. Never store sensitive information like passwords, credit card numbers, or secrets in the payload.

### Signature

The signature ensures the token has not been tampered with:

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

The server uses the secret key to generate the signature when creating the token. On every incoming request, the server recalculates the signature using the same secret and compares it to the signature in the token. If they match, the token is valid.

> [!tip]
> Think of the signature like a wax seal on a letter. Anyone can read the letter (the payload is not encrypted), but only the person with the seal (the secret key) can produce a valid seal. If the seal is broken or different, you know the letter was tampered with.

> [!summary] Section Summary
> A JWT has three Base64Url-encoded parts: the Header (algorithm and type), the Payload (claims about the user), and the Signature (cryptographic proof of integrity). The payload is readable by anyone -- it is encoded, not encrypted.

---

## When to Use JWT

JWT is an excellent choice in certain scenarios and a poor choice in others.

### Good Use Cases

| Scenario | Why JWT Works Well |
|---|---|
| SPAs (React, Angular, Vue) consuming APIs | Client stores the token, sends it with each request |
| Mobile applications | No cookie jar needed, tokens travel in headers |
| Microservices | Each service validates the token independently, no shared session store |
| Third-party API access | Tokens can be scoped and issued to external consumers |
| Cross-domain authentication | Tokens work across origins without CORS cookie issues |

### When NOT to Use JWT

| Scenario | Better Alternative |
|---|---|
| Server-rendered MVC/Razor Pages | [[Cookie Authentication]] with server-side sessions |
| Applications needing instant token revocation | Opaque tokens with a token store |
| Highly sensitive operations requiring per-request validation | Session-based auth with database lookup |

> [!warning] Common Misconception
> JWT is not inherently "more secure" than cookies. It is a different **mechanism** with different tradeoffs. Cookies with `HttpOnly`, `Secure`, and `SameSite` attributes are often more secure for browser-based applications because they are not accessible to JavaScript.

> [!ad-note]
> The primary advantage of JWT is **statelessness**. If your application runs on a single server and serves HTML pages, you gain nothing from JWT and lose the ability to revoke sessions instantly. Use cookies in that case.

> [!summary] Section Summary
> Use JWT for APIs consumed by SPAs, mobile apps, and microservices. Prefer cookie-based authentication for server-rendered applications. JWT shines when statelessness and cross-domain portability matter.

---

## Setting Up JWT Authentication

### Required NuGet Package

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

### Service Registration in Program.cs

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

### Explanation of Each Validation Parameter

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

---

## Token Validation Parameters

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

---

## Generating Tokens

Token generation is the process of creating a JWT to hand to the client after successful authentication.

### Step-by-Step Token Generation

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

### Breakdown of Each Step

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

---

## A Login Endpoint

### Controller-Based Approach

```csharp
[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly TokenService _tokenService;
    private readonly IUserService _userService;

    public AuthController(TokenService tokenService, IUserService userService)
    {
        _tokenService = tokenService;
        _userService = userService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        // Step 1: Validate credentials
        var user = await _userService.ValidateCredentialsAsync(
            request.Email, request.Password);

        if (user is null)
        {
            return Unauthorized(new { message = "Invalid email or password" });
        }

        // Step 2: Generate token
        var token = _tokenService.GenerateToken(
            user.Id.ToString(), user.Email, user.Role);

        // Step 3: Return token
        return Ok(new LoginResponse
        {
            Token = token,
            ExpiresAt = DateTime.UtcNow.AddMinutes(30)
        });
    }
}

public record LoginRequest(string Email, string Password);

public record LoginResponse
{
    public string Token { get; init; } = string.Empty;
    public DateTime ExpiresAt { get; init; }
}
```

### Minimal API Approach

```csharp
app.MapPost("/api/auth/login", async (
    LoginRequest request,
    TokenService tokenService,
    IUserService userService) =>
{
    var user = await userService.ValidateCredentialsAsync(
        request.Email, request.Password);

    if (user is null)
    {
        return Results.Unauthorized();
    }

    var token = tokenService.GenerateToken(
        user.Id.ToString(), user.Email, user.Role);

    return Results.Ok(new
    {
        Token = token,
        ExpiresAt = DateTime.UtcNow.AddMinutes(30)
    });
});
```

> [!danger]
> **Never** return detailed error messages that reveal whether the email or the password was incorrect. Always return a generic "Invalid email or password" to prevent user enumeration attacks.

> [!example]
> **Calling the login endpoint with `curl`:**
> ```bash
> curl -X POST https://localhost:5001/api/auth/login \
>   -H "Content-Type: application/json" \
>   -d '{"email": "john@example.com", "password": "MyP@ssw0rd"}'
> ```
>
> **Response:**
> ```json
> {
>   "token": "eyJhbGciOiJIUzI1NiIs...",
>   "expiresAt": "2026-06-18T15:30:00Z"
> }
> ```

> [!summary] Section Summary
> The login endpoint validates credentials, generates a JWT on success, and returns it to the client. Both controller-based and minimal API approaches follow the same flow: validate, generate, return.

---

## Refresh Tokens

### Why Refresh Tokens Exist

Access tokens should be **short-lived** (15 to 60 minutes). If an access token is stolen, the damage is limited to its short lifetime. But forcing users to re-enter credentials every 30 minutes is unacceptable. Refresh tokens solve this by allowing the client to obtain a new access token without user interaction.

### The Refresh Token Flow

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

### Implementation

#### Refresh Token Entity

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

#### Updated Token Service

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

#### Refresh Endpoint

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

---

## Storing Tokens on the Client

Where you store the JWT on the client side has significant security implications.

| Storage Method | XSS Vulnerable | CSRF Vulnerable | Persists Across Tabs | Persists After Close | Recommendation |
|---|---|---|---|---|---|
| `localStorage` | Yes | No | Yes | Yes | Acceptable for low-risk apps |
| `sessionStorage` | Yes | No | No (per-tab) | No | Slightly better than localStorage |
| HttpOnly Cookie | No | Yes | Yes | Configurable | Best for browser apps (add CSRF protection) |
| In-memory variable | No | No | No | No | Most secure, worst UX |

### localStorage

```javascript
// Store
localStorage.setItem('token', response.token);

// Retrieve and use
fetch('/api/data', {
    headers: { 'Authorization': `Bearer ${localStorage.getItem('token')}` }
});
```

> [!danger]
> `localStorage` is accessible to any JavaScript running on the page. A single XSS vulnerability means an attacker can steal every token stored there.

### HttpOnly Cookie (Recommended for SPAs)

The server sets the token as an HttpOnly cookie:

```csharp
Response.Cookies.Append("access_token", token, new CookieOptions
{
    HttpOnly = true,   // Not accessible to JavaScript
    Secure = true,     // Only sent over HTTPS
    SameSite = SameSiteMode.Strict,  // CSRF protection
    Expires = DateTimeOffset.UtcNow.AddMinutes(30)
});
```

> [!tip]
> The safest pattern for browser-based SPAs is: store the access token in an **HttpOnly cookie** and use `SameSite=Strict` or implement anti-CSRF tokens. This protects against both XSS and CSRF.

> [!summary] Section Summary
> Token storage is a security-critical decision. `localStorage` and `sessionStorage` are vulnerable to XSS. HttpOnly cookies are not accessible to JavaScript but require CSRF protection. For maximum security in browser apps, use HttpOnly cookies with `SameSite=Strict`.

---

## Common JWT Claims

JWT defines a set of **registered claims** (standardized names) and allows **custom claims** for application-specific data.

### Registered (Standard) Claims

| Claim | Full Name | Description | Example |
|---|---|---|---|
| `sub` | Subject | The principal (usually user ID) | `"12345"` |
| `iss` | Issuer | Who issued the token | `"https://myapi.com"` |
| `aud` | Audience | Who the token is intended for | `"https://myapi.com"` |
| `exp` | Expiration Time | Unix timestamp after which the token is invalid | `1718740000` |
| `iat` | Issued At | Unix timestamp when the token was created | `1718736400` |
| `nbf` | Not Before | Unix timestamp before which the token is not valid | `1718736400` |
| `jti` | JWT ID | Unique identifier for the token (prevents replay) | `"a1b2c3d4-..."` |

### Common Custom Claims

| Claim | Description | Example |
|---|---|---|
| `email` | User's email address | `"john@example.com"` |
| `name` | User's display name | `"John Doe"` |
| `role` | User's role for authorization | `"Admin"` |
| `permissions` | Granular permission list | `["read", "write"]` |

> [!tip]
> In ASP.NET Core, use `JwtRegisteredClaimNames` for standard claims and `ClaimTypes` for framework-specific claims like `ClaimTypes.Role`. The `role` claim is special because ASP.NET Core maps it to enable `User.IsInRole()` checks.

> [!summary] Section Summary
> JWT has seven registered claims (`sub`, `iss`, `aud`, `exp`, `iat`, `nbf`, `jti`) and supports arbitrary custom claims. Use registered claims for interoperability and custom claims for application-specific data.

---

## Accessing Claims in Controllers

Once JWT authentication is configured, ASP.NET Core automatically populates `HttpContext.User` with the claims from the validated token.

### Getting a Specific Claim

```csharp
[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile()
{
    // Get the user ID from the "sub" claim
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value
              ?? User.FindFirst(JwtRegisteredClaimNames.Sub)?.Value;

    // Get the email claim
    var email = User.FindFirst(ClaimTypes.Email)?.Value
             ?? User.FindFirst(JwtRegisteredClaimNames.Email)?.Value;

    // Get the role
    var role = User.FindFirst(ClaimTypes.Role)?.Value;

    return Ok(new { userId, email, role });
}
```

### Checking Roles

```csharp
[Authorize]
[HttpDelete("users/{id}")]
public IActionResult DeleteUser(int id)
{
    if (!User.IsInRole("Admin"))
    {
        return Forbid();
    }

    // ... delete logic
    return NoContent();
}
```

### Listing All Claims

```csharp
[Authorize]
[HttpGet("claims")]
public IActionResult GetAllClaims()
{
    var claims = User.Claims.Select(c => new
    {
        Type = c.Type,
        Value = c.Value
    });

    return Ok(claims);
}
```

> [!warning] Common Misconception
> ASP.NET Core maps JWT claim names to its own `ClaimTypes` constants. For example, `sub` becomes `ClaimTypes.NameIdentifier` and `email` becomes `ClaimTypes.Email`. If `User.FindFirst(ClaimTypes.NameIdentifier)` returns null, try the raw JWT claim name. You can disable this mapping:
> ```csharp
> JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Clear();
> ```

> [!example]
> **Using the `[Authorize]` attribute with roles:**
> ```csharp
> [Authorize(Roles = "Admin")]           // Only Admin
> [Authorize(Roles = "Admin,Manager")]   // Admin OR Manager
> ```
> Note: comma-separated roles in a single `[Authorize]` attribute are **OR** logic. For **AND** logic, use multiple attributes:
> ```csharp
> [Authorize(Roles = "Admin")]
> [Authorize(Roles = "Manager")]   // Must be BOTH Admin AND Manager
> ```

> [!summary] Section Summary
> Access JWT claims via `User.FindFirst()`, `User.Claims`, and `User.IsInRole()`. Be aware that ASP.NET Core remaps JWT claim names to `ClaimTypes` constants by default. Use `[Authorize(Roles = "...")]` for declarative role checks.

---

## JWT vs Cookies Comparison

| Aspect | JWT (Bearer Token) | Cookie-Based Sessions |
|---|---|---|
| **Use Case** | APIs, SPAs, mobile apps | Server-rendered web apps |
| **Storage** | Client-side (localStorage, memory, or cookie) | Server-side (memory, database, Redis) |
| **State** | Stateless -- server stores nothing | Stateful -- server maintains session store |
| **CSRF Vulnerability** | Not vulnerable (unless stored in cookies) | Vulnerable -- requires anti-forgery tokens |
| **XSS Vulnerability** | Vulnerable if in localStorage | Not vulnerable if `HttpOnly` cookie |
| **Cross-Domain** | Works naturally via `Authorization` header | Requires complex CORS cookie configuration |
| **Scalability** | Excellent -- no shared state needed | Requires sticky sessions or shared session store |
| **Revocation** | Difficult -- token valid until expiry | Easy -- delete server-side session |
| **Token Size** | Large (claims embedded in every request) | Small (just a session ID cookie) |
| **Server Load** | No database lookup per request | Database/cache lookup per request |

> [!ad-note]
> Neither approach is universally better. Choose based on your architecture. If you are building a monolithic server-rendered app, use cookies. If you are building an API consumed by multiple clients, use JWT.

> [!summary] Section Summary
> JWT and cookies serve different architectural needs. JWT excels in stateless, cross-domain, multi-client scenarios. Cookies excel in server-rendered apps where instant revocation and simpler security (HttpOnly, SameSite) matter.

---

## Security Considerations

### 1. Always Use HTTPS

```
Authorization: Bearer eyJhbGci...
```

Tokens travel in headers. Without HTTPS, anyone on the network can intercept the token (man-in-the-middle attack).

> [!danger]
> **Never** transmit JWTs over plain HTTP. In production, enforce HTTPS at the infrastructure level and set `Secure` on any cookies carrying tokens.

### 2. Keep Tokens Short-Lived

```csharp
expires: DateTime.UtcNow.AddMinutes(15)  // 15 minutes is a good default
```

Short-lived tokens limit the window of exploitation if a token is compromised.

### 3. The Revocation Problem

JWTs cannot be revoked before their expiry. Once issued, they remain valid until `exp`. Mitigation strategies:

- **Short expiration times** -- reduce the damage window
- **Token blacklist** -- store revoked `jti` values in Redis/database (but this sacrifices statelessness)
- **Refresh token revocation** -- revoke the refresh token so no new access tokens can be obtained

### 4. Use Strong Signing Keys

```csharp
// Bad -- too short
"Jwt:Key": "mysecret"

// Good -- at least 256 bits (32+ characters)
"Jwt:Key": "this-is-a-sufficiently-long-secret-key-for-hs256-signing!"
```

### 5. Never Store Sensitive Data in the Payload

The payload is Base64Url encoded, not encrypted. Anyone can decode it:

```bash
echo "eyJzdWIiOiIxMjM0NTY3ODkwIn0" | base64 -d
# {"sub":"1234567890"}
```

> [!warning] Common Misconception
> Developers sometimes assume JWT payloads are encrypted because they "look encrypted." They are not. Base64 is an encoding, not encryption. Anyone with the token string can read every claim.

### 6. Key Rotation

Periodically rotate your signing keys. Use `IssuerSigningKeys` (plural) in `TokenValidationParameters` to accept both old and new keys during the transition period:

```csharp
IssuerSigningKeys = new[]
{
    new SymmetricSecurityKey(Encoding.UTF8.GetBytes(currentKey)),
    new SymmetricSecurityKey(Encoding.UTF8.GetBytes(previousKey))
}
```

### 7. Token Size

Every claim you add increases the token size, and the token is sent with **every request**. Keep the payload lean -- include only what is necessary for authentication and basic authorization. Fetch detailed user data from the API when needed.

> [!tip]
> A rule of thumb: if a claim is needed on nearly every request (like `sub`, `role`, `email`), include it. If it is needed rarely (like a full address or profile picture URL), leave it out and fetch it on demand.

> [!summary] Section Summary
> Key security practices: always use HTTPS, keep tokens short-lived, use strong signing keys, never store secrets in the payload, implement key rotation, and accept that JWTs cannot be easily revoked. Combine short-lived access tokens with revocable refresh tokens for the best balance.

---

## Complete Real-World Example

This section ties everything together into a complete, working JWT authentication setup.

### appsettings.json

```json
{
  "Jwt": {
    "Issuer": "https://myapi.example.com",
    "Audience": "https://myapi.example.com",
    "Key": "a-very-long-secret-key-that-is-at-least-32-characters-long!"
  },
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyApp;Trusted_Connection=true;"
  }
}
```

> [!danger]
> In production, **never** store the JWT signing key in `appsettings.json`. Use environment variables, Azure Key Vault, AWS Secrets Manager, or the .NET Secret Manager (`dotnet user-secrets`).

### Program.cs

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Register custom services
builder.Services.AddScoped<TokenService>();
builder.Services.AddScoped<IUserService, UserService>();

// Configure JWT Authentication
builder.Services.AddAuthentication(options =>
{
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
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
            Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!)),
        ClockSkew = TimeSpan.FromMinutes(1)
    };

    // Optional: handle JWT events for logging/debugging
    options.Events = new JwtBearerEvents
    {
        OnAuthenticationFailed = context =>
        {
            Console.WriteLine($"Authentication failed: {context.Exception.Message}");
            return Task.CompletedTask;
        },
        OnTokenValidated = context =>
        {
            Console.WriteLine($"Token validated for: {context.Principal?.Identity?.Name}");
            return Task.CompletedTask;
        }
    };
});

builder.Services.AddAuthorization();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### AuthController.cs

```csharp
using Microsoft.AspNetCore.Mvc;

[ApiController]
[Route("api/[controller]")]
public class AuthController : ControllerBase
{
    private readonly TokenService _tokenService;
    private readonly IUserService _userService;

    public AuthController(TokenService tokenService, IUserService userService)
    {
        _tokenService = tokenService;
        _userService = userService;
    }

    [HttpPost("login")]
    public async Task<IActionResult> Login([FromBody] LoginRequest request)
    {
        var user = await _userService.ValidateCredentialsAsync(
            request.Email, request.Password);

        if (user is null)
        {
            return Unauthorized(new { message = "Invalid email or password" });
        }

        var accessToken = _tokenService.GenerateAccessToken(
            user.Id.ToString(), user.Email, user.Role);
        var refreshToken = await _tokenService.GenerateRefreshTokenAsync(
            user.Id.ToString());

        return Ok(new
        {
            AccessToken = accessToken,
            RefreshToken = refreshToken.Token,
            ExpiresAt = DateTime.UtcNow.AddMinutes(30)
        });
    }

    [HttpPost("refresh")]
    public async Task<IActionResult> Refresh([FromBody] RefreshRequest request)
    {
        var result = await _tokenService.RefreshAsync(request.RefreshToken);

        if (result is null)
        {
            return Unauthorized(new { message = "Invalid or expired refresh token" });
        }

        return Ok(result);
    }
}

public record LoginRequest(string Email, string Password);
public record RefreshRequest(string RefreshToken);
```

### WeatherController.cs (Protected Endpoint)

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

[ApiController]
[Route("api/[controller]")]
[Authorize]
public class WeatherController : ControllerBase
{
    [HttpGet]
    public IActionResult GetWeather()
    {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        var email = User.FindFirst(ClaimTypes.Email)?.Value;

        var forecast = Enumerable.Range(1, 5).Select(index => new
        {
            Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = "Sunny"
        });

        return Ok(new
        {
            RequestedBy = email,
            Forecast = forecast
        });
    }

    [HttpGet("admin-only")]
    [Authorize(Roles = "Admin")]
    public IActionResult GetAdminData()
    {
        return Ok(new { message = "This data is only visible to admins." });
    }
}
```

### Calling the API

```bash
# Step 1: Login
curl -X POST https://localhost:5001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "john@example.com", "password": "MyP@ssw0rd"}'

# Response:
# {
#   "accessToken": "eyJhbGciOiJIUzI1NiIs...",
#   "refreshToken": "dGhpcyBpcyBhIHJlZn...",
#   "expiresAt": "2026-06-18T15:30:00Z"
# }

# Step 2: Access a protected endpoint
curl https://localhost:5001/api/weather \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..."

# Step 3: Refresh when the access token expires
curl -X POST https://localhost:5001/api/auth/refresh \
  -H "Content-Type: application/json" \
  -d '{"refreshToken": "dGhpcyBpcyBhIHJlZn..."}'
```

> [!example]
> **Testing with Swagger/OpenAPI:**
> If you configured Swagger, click the "Authorize" button in the Swagger UI, enter `Bearer <your-token>`, and all subsequent requests will include the `Authorization` header automatically.

> [!summary] Section Summary
> A complete JWT setup involves configuration in `appsettings.json`, authentication middleware in `Program.cs`, an `AuthController` with login and refresh endpoints, and protected controllers decorated with `[Authorize]`. Clients authenticate by sending `Authorization: Bearer <token>` in request headers.

---

## Related Topics

- [[Authentication Overview]] -- broader look at authentication mechanisms in ASP.NET Core
- [[Cookie Authentication]] -- the traditional server-side session approach, better suited for server-rendered apps
- [[ASP.NET Core Identity]] -- the full-featured identity system that can work alongside JWT
- [[Authorization Policies]] -- fine-grained, policy-based authorization beyond simple role checks

---

## Further Reading

- RFC 7519 -- JSON Web Token specification (https://datatracker.ietf.org/doc/html/rfc7519)
- Microsoft Docs -- Authentication and authorization in ASP.NET Core (https://learn.microsoft.com/en-us/aspnet/core/security/authentication)
- jwt.io -- Interactive JWT debugger and library directory (https://jwt.io)
- OWASP -- JSON Web Token Cheat Sheet (https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
- Auth0 Blog -- JWT Handbook (https://auth0.com/resources/ebooks/jwt-handbook)

---

## Comprehensive Summary

> [!tip] Complete Summary
> **JWT (JSON Web Token)** is a self-contained, stateless authentication mechanism ideal for APIs consumed by SPAs, mobile apps, and microservices. A JWT has three Base64Url-encoded parts -- Header (algorithm), Payload (claims), and Signature (integrity proof) -- separated by dots.
>
> **Setup** in ASP.NET Core involves adding the `Microsoft.AspNetCore.Authentication.JwtBearer` package, configuring `TokenValidationParameters` (issuer, audience, signing key, lifetime), and placing `UseAuthentication()` before `UseAuthorization()` in the middleware pipeline.
>
> **Token generation** creates claims, builds a `JwtSecurityToken` with signing credentials, and serializes it via `JwtSecurityTokenHandler`. **Login endpoints** validate credentials and return the token. **Refresh tokens** (opaque, database-stored) extend sessions without requiring re-authentication -- always implement rotation for security.
>
> **On the client**, HttpOnly cookies are the safest storage for browser apps; `localStorage` is convenient but vulnerable to XSS. **Claims** are accessed in controllers via `User.FindFirst()`, `User.Claims`, and `User.IsInRole()`.
>
> **Security essentials**: always use HTTPS, keep access tokens short-lived (15-30 min), use strong signing keys (256+ bits), never store secrets in the payload, implement key rotation, and understand that JWTs cannot be revoked before expiry -- pair them with revocable refresh tokens.
>
> The fundamental tradeoff: JWT trades **easy revocation** for **stateless scalability**. Choose JWT when you need cross-domain, multi-client, stateless authentication. Choose cookies when you need instant revocation and simpler browser security.
