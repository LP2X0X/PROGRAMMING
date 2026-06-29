---
tags:
  - csharp
  - asp-net-core
  - web-api
  - content-negotiation
  - serialization
---


**Response compression** reduces the size of API responses sent over the network. While not strictly part of content negotiation, it interacts closely with it — the server compresses the serialized response body based on the client's `Accept-Encoding` header.

### How It Works

```http
GET /api/products HTTP/1.1
Accept: application/json
Accept-Encoding: gzip, deflate, br

--- Response ---
HTTP/1.1 200 OK
Content-Type: application/json
Content-Encoding: br
Vary: Accept-Encoding

[compressed body]
```

### Enabling Response Compression

```csharp
using Microsoft.AspNetCore.ResponseCompression;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddResponseCompression(options =>
{
    options.EnableForHttps = true; // Disabled by default for HTTPS (BREACH attack)
    options.Providers.Add<BrotliCompressionProvider>();
    options.Providers.Add<GzipCompressionProvider>();

    // Specify which MIME types to compress
    options.MimeTypes = ResponseCompressionDefaults.MimeTypes.Concat(new[]
    {
        "application/json",
        "application/xml",
        "text/csv",
        "application/x-protobuf"
    });
});

builder.Services.Configure<BrotliCompressionProviderOptions>(options =>
{
    options.Level = System.IO.Compression.CompressionLevel.Optimal;
});

builder.Services.Configure<GzipCompressionProviderOptions>(options =>
{
    options.Level = System.IO.Compression.CompressionLevel.SmallestSize;
});

builder.Services.AddControllers();

var app = builder.Build();

// IMPORTANT: Must be before other middleware that writes responses
app.UseResponseCompression();
app.UseRouting();
app.MapControllers();

app.Run();
```

> [!warning]
> `EnableForHttps` is disabled by default because of the ==BREACH security vulnerability==, which can exploit compression over HTTPS to leak secret tokens. Enable it only if your API does not reflect user input in responses that also contain secrets (e.g., CSRF tokens). Most pure data APIs are safe.

### Compression Levels

| Level | Speed | Compression Ratio | Use Case |
|---|---|---|---|
| `Fastest` | Best | Lowest | Real-time/streaming APIs |
| `Optimal` | Balanced | Good | General-purpose APIs |
| `SmallestSize` | Slowest | Best | Bandwidth-constrained clients |

### Brotli vs Gzip

| Feature | Brotli | Gzip |
|---|---|---|
| Compression ratio | Better (10-25% smaller) | Good |
| Speed | Slower at high levels | Faster |
| Browser support | Modern browsers | Universal |
| Content-Encoding value | `br` | `gzip` |

> [!tip]
> In production, prefer **Brotli** (`br`) with `CompressionLevel.Optimal` as the primary compressor and **Gzip** as the fallback. Brotli provides significantly better compression for JSON payloads.

### Middleware Order

Response compression middleware must be placed ==before== any middleware that writes to the response body:

```csharp
app.UseResponseCompression(); // First
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();
```

> [!ad-note]
> In many production deployments, response compression is handled by the reverse proxy (Nginx, IIS, Cloudflare) rather than ASP.NET Core. This offloads CPU work from the application. If your reverse proxy already compresses, you generally do not need ASP.NET Core's response compression middleware.

> [!summary] Section Summary
> Enable response compression with `AddResponseCompression()` and `UseResponseCompression()`. Configure Brotli and Gzip providers for best coverage. Be aware of the BREACH vulnerability when enabling compression over HTTPS. In production, consider offloading compression to a reverse proxy.
