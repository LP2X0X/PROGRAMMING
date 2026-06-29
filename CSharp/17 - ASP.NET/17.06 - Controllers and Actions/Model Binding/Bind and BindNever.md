---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


These attributes control which properties of a model are eligible for binding, primarily as a defense against **over-posting** (mass assignment) attacks.

### The Over-Posting Problem

Over-posting occurs when a malicious user sends extra form fields that match properties on your model that you did not intend to expose:

```csharp
public class UserProfile
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    public bool IsAdmin { get; set; }   // Should NEVER be set by user input
}

[HttpPost("profile")]
public IActionResult UpdateProfile(UserProfile profile)
{
    // A malicious user sends: Name=Hacker&Email=h@evil.com&IsAdmin=true
    // Without protection, profile.IsAdmin = true!
    _repository.Update(profile);
    return Ok();
}
```

```ad-danger
Over-posting is a real security vulnerability. A classic example is the 2012 GitHub incident where a user exploited mass assignment to add their SSH key to any repository. Always limit which properties can be bound from request data.
```

### [Bind] -- Allow-List Approach

`[Bind]` specifies which properties **are** allowed to be bound. All other properties are ignored:

```csharp
[HttpPost("profile")]
public IActionResult UpdateProfile([Bind("Name,Email")] UserProfile profile)
{
    // Only Name and Email are bound from the request
    // IsAdmin is always false (default) regardless of what the client sends
    
    _repository.Update(profile);
    return Ok();
}
```

### [BindNever] -- Deny-List Approach

`[BindNever]` is applied to model properties to exclude them from binding:

```csharp
using Microsoft.AspNetCore.Mvc.ModelBinding;

public class UserProfile
{
    public string Name { get; set; } = string.Empty;
    public string Email { get; set; } = string.Empty;
    
    [BindNever]
    public bool IsAdmin { get; set; }
    
    [BindNever]
    public DateTime CreatedAt { get; set; }
}
```

### The Better Solution: Separate View Models

The most robust defense against over-posting is to use separate models for input (what the client can send) and domain (what the system manages):

```csharp
// Only contains properties the user is allowed to set
public class UpdateProfileRequest
{
    [Required]
    [StringLength(100)]
    public string Name { get; set; } = string.Empty;
    
    [Required]
    [EmailAddress]
    public string Email { get; set; } = string.Empty;
}

[HttpPost("profile")]
public IActionResult UpdateProfile(UpdateProfileRequest request)
{
    // No risk of over-posting -- the model simply does not have
    // IsAdmin, CreatedAt, or any other sensitive properties
    
    var profile = _repository.GetCurrentUser();
    profile.Name = request.Name;
    profile.Email = request.Email;
    _repository.Update(profile);
    
    return Ok();
}
```

```ad-tip
Prefer using dedicated input models (DTOs/view models) over `[Bind]` or `[BindNever]`. Dedicated models make the contract between client and server explicit, are easier to validate, and cannot accidentally expose new properties when the domain model changes.
```
