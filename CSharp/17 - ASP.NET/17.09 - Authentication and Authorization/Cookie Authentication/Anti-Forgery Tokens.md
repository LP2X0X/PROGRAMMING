---
tags:
  - csharp
  - asp-net-core
  - authentication
  - cookies
  - security
---


Anti-forgery tokens (also called CSRF tokens) are a complementary security mechanism that works alongside cookie authentication to protect against **Cross-Site Request Forgery** attacks.

### The CSRF Problem

When a user is authenticated via cookies, the browser automatically sends those cookies with every request to your domain -- including requests initiated by malicious sites. An attacker can exploit this by tricking the user's browser into sending a request to your server.

### How Anti-Forgery Tokens Work

1. The server generates a unique token and embeds it in the HTML form (as a hidden field) and in a separate cookie.
2. When the form is submitted, the server compares the token from the form field with the token from the cookie.
3. A cross-site attacker cannot read the token from your page (same-origin policy), so they cannot include the correct token in their forged request.

### Usage in Razor Views

```csharp
<!-- The asp-antiforgery tag helper automatically includes the token -->
<form asp-action="Login" asp-controller="Account" method="post">
    <!-- Hidden __RequestVerificationToken field is auto-generated -->
    <input asp-for="Email" />
    <input asp-for="Password" type="password" />
    <button type="submit">Log In</button>
</form>

<!-- Or manually add the token -->
<form method="post" action="/Account/Login">
    @Html.AntiForgeryToken()
    <!-- form fields -->
</form>
```

### Usage in Controllers

```csharp
[HttpPost]
[ValidateAntiForgeryToken]  // Validates the token on POST
public async Task<IActionResult> Login(LoginViewModel model)
{
    // Action code here
}
```

### Global Anti-Forgery Configuration

```csharp
// Apply anti-forgery validation to all POST actions globally
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.Add(new AutoValidateAntiforgeryTokenAttribute());
});
```

> [!example] Why Anti-Forgery Tokens Are Needed Even with SameSite Cookies
> `SameSite` cookies are a strong defense against CSRF, but they are not perfect:
> - Older browsers may not support `SameSite`.
> - `Lax` mode still allows cookies on top-level GET navigations.
> - Defense in depth -- using both `SameSite` and anti-forgery tokens provides layered protection.

> [!summary] Section Summary
> Anti-forgery tokens prevent CSRF attacks by requiring a server-generated token in both a cookie and a hidden form field. Always use `[ValidateAntiForgeryToken]` on POST actions or apply `AutoValidateAntiforgeryTokenAttribute` globally.
