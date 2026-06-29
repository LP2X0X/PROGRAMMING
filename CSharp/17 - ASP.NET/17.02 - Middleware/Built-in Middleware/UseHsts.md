---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseHsts

**`UseHsts`** adds the `Strict-Transport-Security` HTTP response header, which instructs browsers to only access the site over HTTPS for a specified duration. This is called **HTTP Strict Transport Security (HSTS)**.

### How HSTS Works

When a browser receives the `Strict-Transport-Security` header, it:
1. Remembers that this domain requires HTTPS
2. Automatically upgrades all future HTTP requests to HTTPS before sending them
3. Refuses to connect over plain HTTP, even if the user types `http://` in the address bar
4. Prevents users from clicking through certificate warnings for this domain

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddHsts(options =>
{
    options.Preload = true;
    options.IncludeSubDomains = true;
    options.MaxAge = TimeSpan.FromDays(365);
    options.ExcludedHosts.Add("staging.orderportal.com");
});

// Program.cs -- middleware
if (!app.Environment.IsDevelopment())
{
    app.UseHsts();
}

app.UseHttpsRedirection();
```

### Configuration Options

| Option | Default | Description |
|---|---|---|
| `MaxAge` | 30 days | How long browsers remember to use HTTPS |
| `IncludeSubDomains` | `false` | Apply HSTS to all subdomains |
| `Preload` | `false` | Allow inclusion in browser HSTS preload lists |
| `ExcludedHosts` | `localhost`, etc. | Hosts that should not receive the header |

### When You Need It

Any production application served over HTTPS (which should be every application).

### Gotchas

- **Do not use in development** -- browsers cache the HSTS header, and it can make `localhost` inaccessible over HTTP for the duration of `MaxAge`
- Setting `Preload = true` alone does not preload your site -- you must also submit your domain to the browser preload list at `hstspreload.org`
- Start with a short `MaxAge` (e.g., 1 hour) and gradually increase once you confirm HTTPS works correctly across your entire site
- HSTS only takes effect after the **first successful HTTPS response** -- the very first request is still vulnerable to a man-in-the-middle attack (unless preloaded)

> [!summary] Section Summary
> `UseHsts` tells browsers to always use HTTPS for your domain. Configure `MaxAge`, `IncludeSubDomains`, and `Preload` carefully. Only enable it in non-development environments because browsers cache the directive aggressively.
