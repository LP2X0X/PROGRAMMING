---
tags:
  - csharp
  - asp-net-core
  - razor
  - views
  - syntax
---


**Directives** are special Razor instructions that begin with `@` followed by a keyword. They change how the view is parsed or compiled.

### @model -- Declaring the View's Model Type

The most important directive. It tells the view what type `Model` is, enabling IntelliSense and compile-time type checking:

```cshtml
@model ProductDetailViewModel

<h1>@Model.Name</h1>
<p>@Model.Description</p>
<p>Price: @Model.Price.ToString("C")</p>
```

> [!warning] Common Misconception
> `@model` (lowercase) declares the type. `@Model` (uppercase) accesses the instance. Confusing the two is a common error for beginners. `@model` appears once at the top of the file; `@Model` is used throughout to read properties.

### @using -- Importing Namespaces

```cshtml
@using MyApp.Models
@using MyApp.ViewModels
@using System.Globalization

@model ProductDetailViewModel
```

Namespaces added with `@using` in a view apply only to that view. For shared imports, use `_ViewImports.cshtml` (see [[#Shared View Files]]).

### @inject -- Dependency Injection into Views

`@inject` allows you to inject a registered service directly into the view:

```cshtml
@inject IConfiguration Configuration
@inject IStringLocalizer<SharedResource> Localizer

<p>App Version: @Configuration["AppVersion"]</p>
<p>@Localizer["WelcomeMessage"]</p>
```

> [!tip] Practical Tip
> Use `@inject` sparingly. If a view needs complex data that requires service calls, consider using a [[Partial Views and View Components|View Component]] instead. `@inject` is appropriate for simple cross-cutting concerns like localization or configuration values.

### @addTagHelper and @removeTagHelper

These enable or disable [[Tag Helpers]]:

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper MyApp.TagHelpers.*, MyApp
@removeTagHelper Microsoft.AspNetCore.Mvc.TagHelpers.EnvironmentTagHelper, Microsoft.AspNetCore.Mvc.TagHelpers
```

Typically placed in `_ViewImports.cshtml` so all views benefit.

### @tagHelperPrefix

Requires all tag helpers to use a prefix, reducing confusion between tag helpers and plain HTML:

```cshtml
@tagHelperPrefix th:
```

After this directive, you would write `<th:a asp-controller="Home">` instead of `<a asp-controller="Home">`.

### @attribute

Applies a C# attribute to the generated class:

```cshtml
@attribute [Authorize]
```

### @namespace

Overrides the namespace of the generated class (rarely used).

> [!summary] Section Summary
> - `@model` declares the strongly-typed model (lowercase `m` = type, uppercase `M` = instance)
> - `@using` imports namespaces for a single view
> - `@inject` provides dependency injection directly into views (use sparingly)
> - `@addTagHelper` / `@removeTagHelper` control tag helper availability
> - Directives typically go at the top of the file, before any HTML

---
