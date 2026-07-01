---
tags:
  - csharp
  - asp-net-core
  - routing
  - configuration
---


`RouteOptions` is the configuration class that controls **global routing behavior**. You configure it through the options pattern in `Program.cs`:

```csharp
builder.Services.Configure<RouteOptions>(options =>
{
    options.LowercaseUrls = true;
    options.LowercaseQueryStrings = true;
    options.AppendTrailingSlash = false;
});
```

### Available Properties

| Property | Default | Effect |
|---|---|---|
| `LowercaseUrls` | `false` | Generated URLs use lowercase (`/products/details/5` instead of `/Products/Details/5`) |
| `LowercaseQueryStrings` | `false` | Query string keys and values are lowercased in generated URLs |
| `AppendTrailingSlash` | `false` | Appends `/` to generated URLs (`/products/` instead of `/products`) |
| `SuppressCheckForUnhandledSecurityMetadata` | `false` | Suppresses warnings when endpoints have auth metadata but no auth middleware |
| `ConstraintMap` | Built-in constraints | Dictionary mapping constraint names to `IRouteConstraint` types |

### URL Generation Behavior

`LowercaseUrls`, `LowercaseQueryStrings`, and `AppendTrailingSlash` only affect **generated URLs** (from `Url.Action()`, tag helpers, `LinkGenerator`). They do **not** affect how incoming URLs are matched -- routing is always case-insensitive for matching.

```csharp
builder.Services.Configure<RouteOptions>(options =>
{
    options.LowercaseUrls = true;
    options.AppendTrailingSlash = true;
});

// Url.Action("Details", "Products", new { id = 5 })
// Without options: /Products/Details/5
// With options:    /products/details/5/
```

### Registering Custom Route Constraints

The `ConstraintMap` property lets you register [[05 - Custom Route Constraints|custom route constraints]] by name so they can be used in route templates:

```csharp
builder.Services.Configure<RouteOptions>(options =>
{
    options.ConstraintMap.Add("slug", typeof(SlugRouteConstraint));
});

// Now usable in templates:
// [HttpGet("articles/{slug:slug}")]
```

See [[05 - Custom Route Constraints]] for how to implement `IRouteConstraint`.

### LinkOptions -- Per-Call Overrides

`LinkOptions` provides the same URL generation settings as `RouteOptions`, but applied **per call** through `LinkGenerator`:

```csharp
var linkGenerator = app.Services.GetRequiredService<LinkGenerator>();

var url = linkGenerator.GetPathByAction("Details", "Products",
    values: new { id = 5 },
    options: new LinkOptions
    {
        LowercaseUrls = true,
        AppendTrailingSlash = false
    });
```

| Property | Same as RouteOptions? | Effect |
|---|---|---|
| `LowercaseUrls` | Yes | Override casing for this link only |
| `LowercaseQueryStrings` | Yes | Override query string casing for this link only |
| `AppendTrailingSlash` | Yes | Override trailing slash for this link only |

`LinkOptions` values override the global `RouteOptions` defaults for that specific URL generation call. If not set, the global defaults apply.

### Common Production Setup

```csharp
builder.Services.Configure<RouteOptions>(options =>
{
    options.LowercaseUrls = true;
    options.LowercaseQueryStrings = true;
});
```

Most production APIs enable `LowercaseUrls` for consistent, cleaner URLs. `AppendTrailingSlash` is a style preference -- REST APIs typically omit it.

> [!summary] Section Summary
> `RouteOptions` configures global routing behavior: URL casing, trailing slashes, and custom constraint registration. These settings affect URL generation only, not incoming URL matching. Configure via `builder.Services.Configure<RouteOptions>()` in `Program.cs`.
