---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


This section ties everything together into a complete, working JWT authentication setup.

## appsettings.json

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

## Program.cs

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

## AuthController.cs

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

## WeatherController.cs (Protected Endpoint)

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

## Calling the API

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
