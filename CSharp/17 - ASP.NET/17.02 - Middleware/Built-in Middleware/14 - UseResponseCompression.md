---
tags:
  - csharp
  - asp-net-core
  - middleware
  - built-in
---

## UseResponseCompression

**`UseResponseCompression`** compresses HTTP response bodies using algorithms like **gzip** and **Brotli**, reducing the amount of data transferred over the network.

### Configuration

```csharp
// Program.cs -- services
builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true; // see security warning below
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(
        new[] { "application/json", "text/csv" });
});

builder.Services.Configure<BrotliCompressionProvider>(options =>
{
    options.Level = CompressionLevel.Fastest;
});

builder.Services.Configure<GzipCompressionProvider>(options =>
{
    options.Level = CompressionLevel.SmallestSize;
});

// Program.cs -- middleware
app.UseResponseCompression();
// Place BEFORE UseStaticFiles so static responses are also compressed
app.UseStaticFiles();
```

### Compression Levels

| Level | Behavior | Use Case |
|---|---|---|
| `CompressionLevel.Fastest` | Minimal compression, fast | Real-time APIs |
| `CompressionLevel.Optimal` | Balanced | General use |
| `CompressionLevel.SmallestSize` | Maximum compression, slow | Bandwidth-constrained environments |
| `CompressionLevel.NoCompression` | No compression | Debugging |

### When You Need It

When response body size is a concern and you want to reduce bandwidth usage. Particularly useful for JSON API responses and text-heavy content.

### Gotchas

- **BREACH attack risk**: Enabling compression over HTTPS (`EnableForHttps = true`) can expose your application to the BREACH attack, which exploits compression to extract secrets from encrypted responses. Do not enable this if your responses contain sensitive tokens alongside user-controlled content
- Place `UseResponseCompression` **before** middleware that generates responses (like `UseStaticFiles`), because it needs to intercept the response stream before content is written
- Compression does **not** apply to responses that are already compressed (e.g., images like JPEG and PNG)
- Most reverse proxies (Nginx, IIS, Apache) handle compression at the proxy level. Enabling it in ASP.NET Core as well can cause double-compression overhead. Decide where compression should live in your architecture

> [!warning]
> If your reverse proxy already handles compression, **do not enable it again** in ASP.NET Core. Double-compressing wastes CPU and provides negligible size reduction.

> [!summary] Section Summary
> `UseResponseCompression` reduces response size using gzip or Brotli. Configure compression providers and levels based on your performance needs. Be aware of the BREACH attack risk over HTTPS and avoid double-compression when behind a reverse proxy.
