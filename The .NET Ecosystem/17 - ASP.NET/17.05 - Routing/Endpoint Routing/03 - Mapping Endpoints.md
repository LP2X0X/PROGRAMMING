---
tags:
  - csharp
  - asp-net-core
  - routing
  - endpoints
---


The `Map*` methods in `Program.cs` register endpoints with the routing system. Each method targets a different endpoint type.

### `app.MapControllers()`

Registers all controllers that use [[Attribute Routing]]. This is the standard for API controllers:

```csharp
app.MapControllers();
```

This scans all controller classes with `[Route]` or `[Http*]` attributes and registers their routes. It does **not** create conventional routes.

### `app.MapDefaultControllerRoute()`

Registers the standard conventional route for MVC controllers with views:

```csharp
app.MapDefaultControllerRoute();
// Equivalent to:
// app.MapControllerRoute("default", "{controller=Home}/{action=Index}/{id?}");
```

This creates a single conventional route that maps:
| URL | Controller | Action | id |
|---|---|---|---|
| `/` | `Home` | `Index` | `null` |
| `/Products` | `Products` | `Index` | `null` |
| `/Products/Details/5` | `Products` | `Details` | `5` |

### `app.MapControllerRoute()`

Registers a custom conventional route:

```csharp
app.MapControllerRoute(
    name: "blog",
    pattern: "blog/{year:int}/{month:int}/{slug}",
    defaults: new { controller = "Blog", action = "Post" });

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

> [!warning] Common Misconception
> `MapControllers()` and `MapDefaultControllerRoute()` / `MapControllerRoute()` serve different purposes. `MapControllers()` registers **attribute-routed** controllers. `MapDefaultControllerRoute()` and `MapControllerRoute()` register **conventional routes**. If your controller uses `[Route]` attributes, `MapControllerRoute()` will not find it (and vice versa). Most applications call `MapControllers()` for API controllers and optionally `MapDefaultControllerRoute()` for MVC view controllers.

### `app.MapRazorPages()`

Registers all Razor Pages:

```csharp
app.MapRazorPages();
```

Razor Pages use file-system-based routing. A file at `Pages/Products/Index.cshtml` automatically maps to `/Products` (or `/Products/Index`).

### Map Methods Summary

| Method | Endpoint Type | Routing Style |
|---|---|---|
| `MapControllers()` | Controllers with `[Route]`/`[Http*]` | Attribute routing |
| `MapDefaultControllerRoute()` | Controllers without `[Route]` | Conventional (`{controller}/{action}/{id?}`) |
| `MapControllerRoute()` | Controllers without `[Route]` | Custom conventional pattern |
| `MapRazorPages()` | Razor Pages | File-system based |
| `MapGet()`, `MapPost()`, etc. | Minimal API delegates | Inline route template |
| `MapHub<T>()` | SignalR hubs | Hub URL path |
| `MapGrpcService<T>()` | gRPC services | gRPC service path |

> [!summary] Section Summary
> - `MapControllers()` registers attribute-routed controllers (APIs).
> - `MapDefaultControllerRoute()` registers the standard `{controller}/{action}/{id?}` conventional route.
> - `MapControllerRoute()` registers custom conventional routes.
> - `MapRazorPages()` registers Razor Pages with file-system-based routing.
> - Minimal APIs, SignalR, and gRPC have their own `Map*` methods.
> - Do not confuse attribute routing registration (`MapControllers`) with conventional routing registration (`MapControllerRoute`).
