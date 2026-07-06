---
tags:
  - csharp
  - asp-net-core
  - controllers
  - mvc
---


Areas provide a way to organize a large MVC application into smaller functional groupings. Each area has its own set of controllers, views, and models.

### Setting Up an Area

Decorate the controller with the `[Area]` attribute:

```csharp
[Area("Admin")]
public class DashboardController : Controller
{
    public IActionResult Index()
    {
        return View(); // looks in Areas/Admin/Views/Dashboard/Index.cshtml
    }
}

[Area("Admin")]
public class UsersController : Controller
{
    public IActionResult Index()
    {
        return View(); // looks in Areas/Admin/Views/Users/Index.cshtml
    }
}
```

### Folder Structure

```
Project/
  Areas/
    Admin/
      Controllers/
        DashboardController.cs
        UsersController.cs
      Views/
        Dashboard/
          Index.cshtml
        Users/
          Index.cshtml
    Customer/
      Controllers/
        OrdersController.cs
      Views/
        Orders/
          Index.cshtml
```

### Route Configuration

Add an area-aware route **before** the default route:

```csharp
// Program.cs
app.MapControllerRoute(
    name: "areas",
    pattern: "{area:exists}/{controller=Home}/{action=Index}/{id?}");

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");
```

The `{area:exists}` route constraint ensures the segment only matches if an area with that name actually exists.

```ad-info
Areas are primarily used in MVC apps with views. For APIs, it is more common to organize controllers by feature folders or use route prefixes instead of areas.
```
