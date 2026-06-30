---
tags:
  - csharp
  - asp-net-core
  - configuration
  - binding
  - strongly-typed
---


You can use **System.ComponentModel.DataAnnotations** attributes on your options classes to enforce configuration correctness at startup.

### Annotated Options Class

```csharp
using System.ComponentModel.DataAnnotations;

public class SmtpSettings
{
    [Required(ErrorMessage = "SMTP Host is required")]
    public string Host { get; set; } = string.Empty;

    [Range(1, 65535, ErrorMessage = "Port must be between 1 and 65535")]
    public int Port { get; set; }

    [Required]
    [EmailAddress(ErrorMessage = "Username must be a valid email")]
    public string Username { get; set; } = string.Empty;

    [Required]
    [MinLength(8, ErrorMessage = "Password must be at least 8 characters")]
    public string Password { get; set; } = string.Empty;

    public bool UseSsl { get; set; }
}
```

### Commonly Used Data Annotations

| Attribute | Purpose | Example |
|---|---|---|
| `[Required]` | Value must be non-null and non-empty | Connection strings, hostnames |
| `[Range(min, max)]` | Numeric value within bounds | Ports, timeouts, retry counts |
| `[Url]` | Must be a valid URL format | API base URLs |
| `[EmailAddress]` | Must be a valid email format | Notification addresses |
| `[MinLength(n)]` | String minimum length | Passwords, API keys |
| `[MaxLength(n)]` | String maximum length | Short codes, identifiers |
| `[RegularExpression]` | Must match a regex pattern | Custom format constraints |
| `[StringLength(max)]` | String length within bounds | General text fields |

> [!example] Annotated URL Configuration
> ```csharp
> public class ApiEndpointSettings
> {
>     [Required]
>     public string Name { get; set; } = string.Empty;
> 
>     [Required]
>     [Url(ErrorMessage = "BaseUrl must be a valid URL")]
>     public string BaseUrl { get; set; } = string.Empty;
> 
>     [Range(1, 300, ErrorMessage = "Timeout must be between 1 and 300 seconds")]
>     public int TimeoutSeconds { get; set; } = 30;
> }
> ```

> [!summary] Section Summary
> Data annotations from `System.ComponentModel.DataAnnotations` can be placed on options class properties. They are only enforced when paired with `ValidateDataAnnotations()` during registration (covered next).
