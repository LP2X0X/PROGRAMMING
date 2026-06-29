---
tags:
  - csharp
  - asp-net-core
  - authentication
  - jwt
  - api
  - security
---


## Controller-Based Approach

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

## Minimal API Approach

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
