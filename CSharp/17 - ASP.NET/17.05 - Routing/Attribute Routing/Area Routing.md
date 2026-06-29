---
tags:
  - csharp
  - asp-net-core
  - routing
  - attribute-routing
---


**Areas** are a way to organize a large ASP.NET Core MVC application into functional groups, each with its own controllers, views, and models.

### Declaring an Area

```csharp
[Area("Admin")]
[Route("admin/[controller]")]
public class DashboardController : Controller
{
    [HttpGet]
    public IActionResult Index() => View();
    // Matches: GET admin/Dashboard
}
```

### The `[area]` Token

The `[area]` token in route templates resolves to the area name:

```csharp
[Area("Admin")]
[Route("[area]/[controller]")]
public class DashboardController : Controller
{
    [HttpGet("[action]")]
    public IActionResult Index() => View();
    // Matches: GET Admin/Dashboard/Index
}
```

### Area Routing with Conventional Routes

Areas also work with conventional routing:

```csharp
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");
```

The `{area:exists}` constraint ensures the area segment matches a registered area.

> [!summary] Section Summary
> - Areas organize large applications into functional groups with isolated controllers and views.
> - `[Area("Name")]` declares a controller's area; `[area]` token resolves to it in route templates.
> - Conventional routes support areas with the `{area:exists}` constraint.
