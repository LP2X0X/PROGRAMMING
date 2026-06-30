---
tags:
  - csharp
  - asp-net-core
  - model-binding
  - controllers
---


```ad-summary
**Model binding** automatically maps HTTP request data to action method parameters and model properties, eliminating manual extraction from `Request.Query`, `Request.Form`, and `Request.RouteValues`.

**Key points:**

- The default binding source priority is: Form values, Route values, Query string. Headers and Body require explicit attributes.
- Explicit source attributes (`[FromBody]`, `[FromRoute]`, `[FromQuery]`, `[FromHeader]`, `[FromForm]`, `[FromServices]`) override default priority and should be used for clarity.
- Simple types are converted from strings using `TypeConverter`. Conversion failures populate `ModelState` errors.
- Complex types are recursively bound property-by-property, with support for nested objects via dot notation.
- `[ApiController]` changes inference: complex types default to `[FromBody]`, simple types to `[FromRoute]` or `[FromQuery]`.
- Collections (arrays, lists, dictionaries) bind from repeated query parameters or indexed form fields.
- File uploads use `IFormFile` / `IFormFileCollection` with `multipart/form-data` encoding.
- Custom model binders implement `IModelBinder` for scenarios the built-in binders do not cover.
- `[BindProperty]` opts controller/PageModel properties into binding; `SupportsGet = true` enables GET binding.
- Protect against over-posting with `[Bind]`, `[BindNever]`, or (preferred) dedicated input DTOs.
```

## Related Topics

- [[Controllers Overview]] -- controller fundamentals and conventions
- [[Action Results]] -- returning responses from actions
- [[Validation]] -- validating bound models with data annotations and custom validators
- [[Filters]] -- action filters that run before/after model binding
- [[17.05 - Routing]] -- how requests are matched to actions before binding occurs
