---
tags:
  - csharp
  - asp-net-core
  - validation
  - model-binding
---


ASP.NET Core MVC supports **client-side validation** via the **jQuery Unobtrusive Validation** library. This prevents form submission before the request even reaches the server, providing immediate feedback to users.

### How It Works

1. Tag Helpers generate `data-val-*` HTML attributes from your Data Annotations
2. The jQuery Unobtrusive Validation library reads these attributes at runtime
3. When the user submits the form, client-side validation runs first
4. If validation fails, the form does not submit and error messages appear next to the fields

### Required Scripts

Include these scripts (typically via a partial or layout):

```html
<script src="~/lib/jquery/dist/jquery.min.js"></script>
<script src="~/lib/jquery-validation/dist/jquery.validate.min.js"></script>
<script src="~/lib/jquery-validation-unobtrusive/jquery.validate.unobtrusive.min.js"></script>
```

### Razor Form with Tag Helpers

```csharp
@model CreateProductRequest

<form asp-action="Create" asp-controller="Products" method="post">
    <!-- Validation summary for model-level errors -->
    <div asp-validation-summary="ModelOnly" class="text-danger"></div>

    <div class="form-group">
        <label asp-for="Name"></label>
        <input asp-for="Name" class="form-control" />
        <span asp-validation-for="Name" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="Price"></label>
        <input asp-for="Price" class="form-control" />
        <span asp-validation-for="Price" class="text-danger"></span>
    </div>

    <div class="form-group">
        <label asp-for="ContactEmail"></label>
        <input asp-for="ContactEmail" class="form-control" />
        <span asp-validation-for="ContactEmail" class="text-danger"></span>
    </div>

    <button type="submit" class="btn btn-primary">Create</button>
</form>
```

The generated HTML includes validation metadata:

```html
<input class="form-control" type="text" id="Name" name="Name"
    data-val="true"
    data-val-required="The Name field is required."
    data-val-length="The Name must be between 3 and 200 characters."
    data-val-length-max="200"
    data-val-length-min="3" />
<span class="text-danger field-validation-valid"
    data-valmsg-for="Name"
    data-valmsg-replace="true"></span>
```

### Validation Summary Modes

The `asp-validation-summary` tag helper has three modes:

| Mode | Behavior |
|---|---|
| `None` | No summary displayed |
| `ModelOnly` | Shows only model-level errors (key = `string.Empty`) |
| `All` | Shows all errors -- model-level and property-level |

```ad-danger
Client-side validation is a **UX convenience**, not a security measure. It can be bypassed by disabling JavaScript, using browser dev tools, or calling your API directly with cURL/Postman. **Always validate on the server.** Client-side validation reduces unnecessary round trips but must never be your only line of defense.
```

```ad-summary
jQuery Unobtrusive Validation reads `data-val-*` attributes generated from Data Annotations to provide instant client-side feedback. Use `asp-validation-for` for per-field errors and `asp-validation-summary` for model-level errors. Client-side validation is a UX feature -- always validate server-side as well.
```
