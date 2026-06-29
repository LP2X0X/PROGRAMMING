---
tags:
  - csharp
  - asp-net-core
  - razor-pages
  - views
---


Razor Pages uses **convention-based routing** where the file path under `/Pages/` directly maps to the URL:

| File Path | URL |
|---|---|
| `/Pages/Index.cshtml` | `/` or `/Index` |
| `/Pages/Privacy.cshtml` | `/Privacy` |
| `/Pages/Products/Index.cshtml` | `/Products` or `/Products/Index` |
| `/Pages/Products/Create.cshtml` | `/Products/Create` |
| `/Pages/Products/Edit.cshtml` | `/Products/Edit` |
| `/Pages/Account/Login.cshtml` | `/Account/Login` |

**Key conventions:**
- The `/Pages/` folder is the root (not included in the URL)
- `Index.cshtml` is the default document for a folder
- Folder hierarchy = URL hierarchy
- The `.cshtml` extension is not included in the URL

### Changing the Pages Root Folder

```csharp
builder.Services.AddRazorPages().AddRazorPagesOptions(options =>
{
    options.RootDirectory = "/Content";  // Instead of /Pages
});
```

### Adding Area-Based Razor Pages

```csharp
builder.Services.AddRazorPages().AddRazorPagesOptions(options =>
{
    options.Conventions.AuthorizeAreaFolder("Admin", "/");
});
```

> [!summary] Section Summary
> - File path under `/Pages/` directly determines the URL (convention-based routing)
> - `Index.cshtml` serves as the default document for its folder
> - No explicit route registration is needed -- the file system IS the routing table
> - The root folder can be customized, and areas are supported

---
